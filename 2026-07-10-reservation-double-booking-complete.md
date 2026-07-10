# 2026-07-10 TIL — 예약 시스템(더블부킹 방지), 13개 테이블 완성

> CoFit 백엔드의 하이라이트인 PT 예약 시스템을 구현하고,
> 남은 테이블(availability, centers, auth_logs, notifications)까지 채워
> **스키마 13개 테이블 전체를 API로 완성**했다.
> 오늘의 핵심은 "동시성/더블부킹을 DB 레벨에서 막는 법".

---

## 1. 오늘 완성한 것 — 13개 테이블 전부

이제 스키마의 모든 테이블이 API로 구현됨:
- users, body_records, diets, user_goals, user_exercise_goals,
  trainer_feedbacks, pt_packages, **pt_reservations**,
  trainer_availability, centers, auth_logs, notifications
- (goal_categories는 조회 전용 Entity)

---

## 2. ★핵심★ 더블부킹 방지 — 동시성을 DB로 해결

### 문제
트레이너 A의 화요일 3시 슬롯에 회원 둘이 **동시에** 예약을 누르면?
애플리케이션 사전 체크만으론 둘 다 "빈 슬롯"으로 보여서 둘 다 통과할 수 있다.

### 해결: 조건부 UNIQUE 생성 컬럼 (DB 레벨 방어)
```sql
active_booking_key VARCHAR(50) AS (
    CASE WHEN status IN ('RESERVED','COMPLETED')
         THEN CONCAT(trainer_id, '_', scheduled_at)
         ELSE NULL END
) STORED,
UNIQUE KEY uq_active_booking (active_booking_key)
```
- 활성 상태(RESERVED/COMPLETED)일 때만 key 값이 생성됨.
- 그 key에 UNIQUE → 같은 트레이너·같은 시간은 **DB가 하나만 허용**.
- 취소(CANCELED)/노쇼(NOSHOW)면 key가 NULL → **슬롯이 다시 열림**.

### 애플리케이션에서 UNIQUE 위반을 409로 변환
```java
try {
    PtReservation saved = reservationRepository.saveAndFlush(reservation);
    return PtReservationResponse.from(saved);
} catch (DataIntegrityViolationException e) {
    throw new DuplicateResourceException("그 시간대는 이미 예약되어 있습니다.");
}
```
- **saveAndFlush**: 즉시 DB에 반영해서 UNIQUE 위반을 바로 감지.
- **DataIntegrityViolationException**: DB 제약 위반 시 Spring이 던지는 예외 → 409로 변환.
- 즉 "애플리케이션 사전 체크 + DB 최종 방어"의 이중 구조.

### 배운 것
- 동시성 문제는 애플리케이션 검증만으론 불완전하다.
  **DB의 UNIQUE 제약이 최후의 방어선.**
- "조건부 UNIQUE"(생성 컬럼)로 "활성 예약만 유일, 취소하면 재개방"을 자동화.

---

## 3. 두 테이블 연계 트랜잭션 (예약 ↔ 패키지)

```java
// 예약 시: 패키지 남은횟수 차감
pkg.useSession();          // remaining_sessions--  (0이면 예외)

// 취소 시: 남은횟수 복구
r.cancel();                // status = CANCELED → key NULL → 슬롯 재개방
pkg.restoreSession();      // remaining_sessions++
```
- 하나의 @Transactional 안에서 두 테이블(reservation, package)이 함께 변경.
- 예약 성공하면 횟수 차감, 취소하면 복구 → 데이터 일관성 유지.
- 상태 변경은 setter가 아니라 의도가 드러나는 메서드(useSession/restoreSession).

---

## 4. 상태 기반 검증

```java
public boolean isCancelable() {
    return "RESERVED".equals(this.status);  // 이미 완료된 건 취소 불가
}
```
- 상태에 따라 허용 동작이 달라진다. 완료(COMPLETED)된 예약은 취소 못 함.
- "지금 상태에서 이 동작이 가능한가"를 메서드로 캡슐화.

---

## 5. 불변 엔티티 (auth_logs)

- 로그인 기록은 **수정/삭제 메서드를 아예 안 만든다.**
- 이유: 로그는 "일어난 사실의 기록". 나중에 성공→실패로 바꾸면 조작이 됨.
- **update API를 안 만드는 것도 설계다.** (데이터 신뢰성/감사 목적)
- login_status는 상수 Set으로 검증:
  ```java
  private static final Set<String> VALID_STATUS = Set.of("SUCCESS","FAIL","LOGOUT");
  if (!VALID_STATUS.contains(request.loginStatus())) throw ...;
  ```
  → 문자열을 코드 여기저기 흩뿌리지 않고 한 곳에 모음.

---

## 6. 계층형 아키텍처 정리 (오늘 확실히 이해)

CRUD 하나에 파일 5개가 나오는 이유 = **각 파일이 한 가지 역할만**:

| 파일 | 역할 | 비유 |
|---|---|---|
| Controller | HTTP 요청/응답 | 홀 직원 |
| Service | 비즈니스 로직·검증 | 주방장 |
| Repository | DB 접근 | 창고 담당 |
| Entity | DB 매핑 데이터 | 레시피 |
| DTO | 주고받는 그릇 | 주문서/접시 |

- **관심사 분리**: 하나 바꿀 때 한 곳만 고침 (DB 바꿔도 Repository만).
- **테스트 용이**: Service만 떼서 로직 검증 가능.
- **보안**: DTO로 Entity 노출 격리 (id 등 조작 방지).
- **협업**: 파일 나뉘어 있어야 여러 명이 동시 작업.

---

## 7. 전체 여정에서 쌓인 패턴 (13개 만들며 반복)

- 생성자 주입, @Transactional(readOnly=true) 기본 + 쓰기만 재정의
- 검증 순서: 존재(404) → 자격(400) → 중복(409)
- Entity에 setter 대신 의도 드러나는 메서드(markChecked/cancel/markAsRead...)
- 변경 감지(dirty checking) — save() 없이 자동 UPDATE
- DTO 분리(Request/Response), Entity를 컨트롤러로 안 넘김
- findXOrThrow() 중복 제거
- 상태 전이 캡슐화, 두 테이블 연계 트랜잭션

---

## 8. 오늘의 교훈

- **동시성/무결성은 애플리케이션 + DB 이중으로 방어한다.**
  특히 더블부킹 같은 건 DB의 UNIQUE 제약이 최종 보루.
- **API를 "안 만드는 것"도 설계다.** (로그의 수정/삭제 배제)
- 13개 테이블을 반복하며 **같은 패턴을 몸에 익혔다.**
  이제 새 도메인이 와도 구조는 자동으로 나온다 — "구조 재사용, 규칙은 도메인마다 판단".

---

## 면접 카드 (오늘 확보)

- "중복 예약/동시성을 어떻게 막았나요?"
  → 애플리케이션 사전 체크 + DB 조건부 UNIQUE 생성 컬럼으로 이중 방어.
    활성 상태일 때만 key가 생겨 동시 요청 시 DB가 하나만 허용, 취소하면
    key가 NULL이 되어 슬롯 재개방. 위반은 DataIntegrityViolationException을
    잡아 409로 변환.
- "계층형 아키텍처를 왜 쓰나요?"
  → 각 계층이 단일 책임(HTTP/로직/DB)을 갖게 해 유지보수·테스트·협업·보안을 확보.
- "수정 API를 왜 어떤 엔티티엔 안 만드셨나요?"
  → 로그처럼 기록 성격의 데이터는 불변으로 두어 감사 기록 조작을 방지.

---

## 다음
- README 작성 (프로젝트 소개 + ERD + 기술스택 + API 목록 + 하이라이트).
- 만든 것을 "보여줄 수 있는 상태"로 포장하는 게 다음 우선순위.
