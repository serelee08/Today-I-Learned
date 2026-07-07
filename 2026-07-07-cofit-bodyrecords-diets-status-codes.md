# 2026-07-07 TIL — CoFit body_records / diets 세로줄, 상태코드 설계, ERD

> CoFit(AI PT 관리 SaaS) 백엔드에 body_records(체중)와 diets(식단) 
> users 패턴 복제로 추가. 핵심은 "같은 패턴이라도 도메인마다 규칙이 다르다"는 것.

---

## 1. 오늘 구현한 것

- **body_records (체중 기록)** 세로줄 6개 파일 — Entity/Repository/DTO×2/Service/Controller
- **diets (식단 기록)** 세로줄 6개 파일
- **GlobalExceptionHandler에 409(중복) 핸들러 추가**
- **전체 13개 테이블 ERD 작성** (PNG)

→ CoFit 백엔드 현황: 13개 테이블 중 **users / body_records / diets 3개** 구현 완료.

---

## 2. 같은 패턴, 다른 규칙 — 도메인마다 검증이 다르다

users를 복붙해서 만들었지만, 도메인 규칙이 달라서 검증이 각각 달랐다.

| | body_records | diets |
|---|---|---|
| 하루 기록 횟수 | **1회만** (UNIQUE) | 여러 번 OK (아침·점심·저녁) |
| 중복 시 | 409 Conflict | 제약 없음 |
| 상태 필드 | 없음 | feedback_status (PENDING→CHECKED) |
| AI 필드 | 없음 | calories/탄단지 (처음 null, 나중에 AI가 채움) |

**교훈**: 복붙은 구조를 재사용하는 것이지 규칙까지 복사하는 게 아니다.
같은 CRUD 패턴이라도 **비즈니스 규칙은 도메인에 맞게 판단**해야 한다.

---

## 3. HTTP 상태코드 구분 — 이게 REST 성숙도

오늘 처음으로 3가지 상태코드를 상황에 맞게 나눴다.

- **400 Bad Request** — 입력이 틀림 (role이 T/M 아님, 회원만 가능한데 트레이너가 시도)
- **404 Not Found** — 리소스 없음 (없는 회원/기록 조회)
- **409 Conflict** — 입력은 정상인데 이미 존재해서 충돌 (같은 날 체중 2번)

### 검증 순서 원칙 (Service에서)
```
① 존재 확인 (404) → ② 자격 확인 (400) → ③ 중복 확인 (409)
```
싼 검증부터, 논리 순서대로. 없는 회원인데 중복부터 확인하면 낭비.

### 오늘 실수로 배운 것
`GET /api/diets`를 memberId 없이 호출 → 500 에러.
- 원인: `@RequestParam Long memberId`가 필수인데 안 보냄
- 근데 이건 클라이언트 잘못이라 **500이 아니라 400이 맞다**
- `@ExceptionHandler(Exception.class)`가 다 500으로 잡아버려서 생긴 문제
- 해결: `MissingServletRequestParameterException` 핸들러 추가 → 400으로 정확히

**교훈**: 상태코드는 "책임 소재"를 나타낸다. 500=서버 잘못, 400=클라이언트 잘못.
파라미터 누락을 500으로 내보내면 API 쓰는 사람이 "서버 버그인가?" 헷갈린다.

---

## 4. JPA 변경 감지 (dirty checking) 재확인

```java
public void markChecked(Long id) {
    Diet diet = findDietOrThrow(id);
    diet.markChecked();   // save() 호출 안 함!
}
```
- `@Transactional` 안에서 Entity 값을 바꾸면, 트랜잭션 종료 시 JPA가
  자동으로 UPDATE 쿼리를 날린다. `save()` 명시 호출 불필요.
- MyBatis엔 없는 개념. JPA를 쓴다는 건 이걸 이해하고 쓴다는 것.

---

## 5. Entity에 setter 대신 의도가 드러나는 메서드

```java
// ❌ setter를 다 열면 아무 데서나 상태가 바뀜 → 추적 불가
// ✅ 의도가 드러나는 메서드로만 상태 변경 허용
public void markChecked() { this.feedbackStatus = "CHECKED"; }
public void applyNutritionAnalysis(...) { ... }   // AI 결과 채우기
```
- 상태 전이를 메서드로 캡슐화 → "어떤 변경이 허용되는지"를 코드로 강제.
- 나중에 "CHECKED에서 PENDING으로 못 돌아가게" 같은 규칙 넣기도 쉬움.

---

## 6. FK 무결성 애플리케이션 방어

body_records/diets는 users를 참조(member_id)하므로,
저장 전에 "그 회원이 진짜 존재하나?"를 확인:
```java
User member = userRepository.findById(request.memberId())
    .orElseThrow(() -> new ResourceNotFoundException(...));  // 없으면 404
```
- DB FK 제약도 막아주지만, 미리 확인해 **친절한 404 메시지**를 주는 게 실무.
- FK 위반으로 터지면 사용자는 "무슨 에러?" → 명확한 메시지가 낫다.

---

## 7. BigDecimal (돈·측정값)

- 체중·영양소는 `double` 아닌 `BigDecimal`.
- double은 부동소수점 오차(0.1+0.2 = 0.30000...4). 정확해야 하는 값엔 금지.
- 스키마 DECIMAL(5,2) ↔ 자바 BigDecimal precision/scale로 매핑.

---

## 8. ERD 작성

- 13개 테이블 전체 관계도를 작성.
- 주목할 설계 3가지 (면접 카드):
  1. **users 셀프 참조 FK** — 트레이너/회원을 한 테이블에, trainer_id로 연결
  2. **pt_reservations.active_booking_key** — 조건부 UNIQUE 생성컬럼으로 더블부킹 차단
  3. **trainer_feedbacks.diet_id FK+UK** — 식단 1건당 피드백 1건(1:1) 강제

---

## 9. 오늘의 교훈

- **복붙은 구조 재사용이지 규칙 복사가 아니다.** 도메인마다 검증을 다시 판단.
- **상태코드는 책임 소재를 표현한다.** 400/404/409/500을 정확히 구분해야 REST 성숙.
- **에러 메시지를 위에서부터 정독하면 원인이 좁혀진다.** (memberId 누락 → 메시지가 직접 알려줌)
- **코드가 돌아가는 것 ≠ 데이터가 올바른 것.** 무결성 검증까지가 진짜 완성.

---

## 다음 (미구현)
user_goals, user_exercise_goals, trainer_feedbacks,
trainer_availability, pt_packages, pt_reservations, auth_logs, notifications
