# 2026-07-04 TIL — CoFit 백엔드 JPA 셋업 & 실전 디버깅

> CoFit(AI PT 관리 SaaS)의 `users` API를 Spring Boot + JPA로 처음부터 띄우면서
> 겪은 에러들과 해결 과정. 오늘의 핵심은 "코드 작성"이 아니라 "실전 디버깅"이었다.

---

## 1. JPA 기본 개념 정리 (MyBatis와 비교)

- **MyBatis**: SQL을 XML에 직접 작성. DAO + Mapper.xml 구조.
- **JPA**: SQL 자동 생성. `Entity` + `Repository` 구조로 SQL 없이 CRUD.
  - `JpaRepository<User, Long>` 상속만으로 findById/findAll/save/delete 자동 제공.
  - 커스텀 쿼리는 메서드 이름으로 자동 생성 (예: `findByTrainerCode`).

### JPA 실무 패턴 (면접 포인트)
- **`@Transactional(readOnly = true)`를 클래스 기본**으로 두고, 쓰기 메서드만 `@Transactional` 재정의 → 조회 성능 최적화.
- **변경 감지(dirty checking)**: 트랜잭션 안에서 Entity 필드를 바꾸면
  `save()`를 호출하지 않아도 커밋 시점에 자동 UPDATE. MyBatis엔 없는 개념.
- **Entity에 setter를 열지 않는다**: 생성은 `@Builder`, 수정은 의도가 드러나는
  메서드(`updateProfile`)로만. 무분별한 상태 변경을 막는 실무 패턴.

---

## 2. 에러 #1 — Hibernate가 스키마 생성 컬럼을 못 읽음

### 증상
```
Unknown column 'RESERVED' in 'WHERE'
Unable to determine Dialect without JDBC metadata
```

### 원인 (로그를 위에서부터 읽어서 발견)
- 맨 아래 `Dialect` 에러는 결과일 뿐, 진짜 원인은 로그 중간의
  `Unknown column 'RESERVED'`였다.
- `pt_reservations` 테이블의 **생성 컬럼(generated column)** 안에 있던
  `CASE WHEN status IN ('RESERVED', ...)` 의 문자열 `'RESERVED'`를
  Hibernate가 부팅 시 메타데이터를 읽으며 **컬럼 이름으로 오해**해서 발생.

### 해결
`application.properties`에 Dialect를 명시해서 메타데이터 자동탐색을 우회:
```properties
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
```

### 배운 것
- **에러는 연쇄한다. 스택트레이스의 맨 아래(최종 에러)가 아니라
  로그 위쪽의 첫 실패 지점(root cause)을 찾아야 한다.**
- `ddl-auto=validate`: 내가 SQL로 만든 테이블을 JPA가 건드리지 않고
  Entity와 일치하는지 검증만 하는 안전한 설정. (create/update는 테이블을 변경하므로 위험)

---

## 3. 에러 #2 — 포트 충돌 (Port 8080 already in use)

### 증상
```
APPLICATION FAILED TO START
Web server failed to start. Port 8080 was already in use.
```

### 원인
- 이미 실행 중이던 `hospital-reservation-backend`(CatchCare)가 8080을 점유.
- 한 포트는 한 번에 하나의 프로세스만 사용 가능.

### 해결
- Boot Dashboard에서 안 쓰는 앱 종료, 또는 `server.port=8081`로 분리.
- 터미널: `lsof -i :8080` → `kill -9 [PID]`

### 배운 것 (포트 개념)
- 포트는 컴퓨터에 고정된 숫자가 아니라 **앱이 실행 시 점유하는 통신 창구 번호**.
- 0~1023은 예약 포트(80=HTTP, 3306=MySQL 등)라 피하고 8080번대 사용.
- 여러 앱 동시 실행 시 각각 다른 포트 할당. (마이크로서비스의 기본 원리)
- 앱 포트(`server.port`)와 DB 포트(`datasource.url`의 3306)는 별개.

---

## 4. 데이터 무결성 — role별 조건부 검증

### 문제 인식
- `users` 테이블은 트레이너(T)와 회원(M)을 한 테이블에 담음.
- `trainer_id`, `center_id`를 무조건 null 허용/불허로 두면 안 됨:
  - 회원(M)인데 담당 트레이너 없음 → 잘못된 데이터
  - 트레이너(T)인데 담당 트레이너 있음 → 잘못된 데이터
- 즉 **컬럼은 nullable이되, role에 따라 null 여부가 결정**되는 조건부 규칙.

### 해결 (Service 계층 검증)
```java
if ("M".equals(request.role()) && request.trainerId() == null) {
    throw new IllegalArgumentException("회원은 담당 트레이너가 반드시 필요합니다.");
}
if ("T".equals(request.role()) && request.trainerId() != null) {
    throw new IllegalArgumentException("트레이너는 담당 트레이너를 가질 수 없습니다.");
}
```
- Postman 테스트로 양방향 400 확인 완료.

### 배운 것
- 검증(validation)은 **비즈니스 규칙**이므로 Controller가 아니라 **Service**에 둔다.
- 검증은 데이터가 DB에 닿기 전에 막는 게 원칙. (잘못된 데이터 저장 후 고치는 것보다 100배 싸다)
- **프론트 검증 = UX 편의, 백엔드 검증 = 진짜 방어.** 클라이언트는 신뢰하지 않는다(Never trust the client).
  프론트에서 폼을 숨겨도 API 직접 호출로 우회 가능하므로 백엔드에서 반드시 재검증.
- 같은 규칙이 DB(CHECK) / 백엔드(Service) / 프론트(폼)에 일관되게 표현되어야 한다.(계층적 방어)

---

## 5. 에러 #3 — git push rejected (fetch first)

### 증상
```
! [rejected] main -> main (fetch first)
Updates were rejected because the remote contains work that you do not have locally.
```

### 원인
- GitHub에서 레포 생성 시 README를 체크 → 원격에 커밋이 이미 존재.
- 로컬 커밋과 원격 커밋의 히스토리 뿌리가 달라서 push 거부됨 (git의 안전장치).

### 해결
```bash
git pull origin main --allow-unrelated-histories
git push -u origin main
```
- `--allow-unrelated-histories`: 서로 다른 뿌리의 히스토리 병합을 허용.

### 배운 것 (재발 방지)
- **다음부터 GitHub 레포는 README/gitignore/license 체크 없이 "빈 레포"로 생성**하면
  이 에러 자체가 안 난다.
- `--force`는 원격을 로컬로 덮어쓰므로 팀 작업을 날릴 수 있다. 개인 레포 초기화 때만 신중히.

---

## 6. 에러 #4 — 잘못된 DB 조회

### 증상
```
Table 'hospital_reservation.users' doesn't exist
```

### 원인
- DB 도구(DBeaver)의 현재 선택 스키마가 `hospital_reservation`(CatchCare)로 잡혀 있었음.
- CoFit의 `users`는 `cofit` 데이터베이스에 있는데 엉뚱한 DB에서 조회.

### 해결
```sql
USE cofit;
SELECT * FROM users;
```

### 배운 것
- 여러 DB를 다룰 때 **"지금 어느 스키마에 연결돼 있는지"를 항상 인지**해야 한다.
- 쿼리 앞에 `USE [db];`를 명시하는 습관 → 실수로 다른 DB(특히 운영)에 날리는 사고 방지.

---

## 7. 오늘의 결과

- CoFit `users` CRUD API를 Spring Boot + JPA로 처음부터 띄우는 데 성공.
- `POST /api/users` → 201 + trainerCode 자동생성 확인.
- DB에 실제 저장 확인 (id=1, 김트레이너, TR-9CF63C).
- role별 무결성 검증(양방향 400) 확인.
- GitHub `cofit-backend` 레포에 push.

## 8. 오늘의 핵심 교훈

- **오늘 시간의 대부분은 코드 작성이 아니라 디버깅이었다.**
  포트 충돌, Dialect, DB 선택, git 히스토리 — 전부 "코드 밖"의 문제.
  실무 개발의 상당 부분이 이런 환경/설정/도구 문제 해결이라는 걸 체감.
- **에러 메시지를 위에서부터 정독하면 원인이 좁혀진다.**
  추측하기 전에 로그가 이미 답을 갖고 있는 경우가 많다.
- **검증된 선택 하나로 빨리 실행하는 것 > 완벽한 선택을 여러 번 고민하는 것.**
