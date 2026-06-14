# TIL - 2026.06.14

## 오늘 한 것
Hospital Reservation System 백엔드 CRUD 완성
(patient, doctor, appointment, medical_record) + DB 연동 + Postman 테스트

---

## 1. MyBatis 카멜케이스 매핑 트러블슈팅

### 문제
Postman GET 요청 시 `departmentName`, `departmentNo`, `createdAt`이 null로 반환됨.
`description`만 정상 출력.

### 원인
DB 컬럼은 snake_case(`department_name`), Java 필드는 camelCase(`departmentName`).
언더스코어가 있는 컬럼만 매핑 실패.

### 해결
application.properties에 추가:
mybatis.configuration.map-underscore-to-camel-case=true

---

## 2. MariaDB + MySQL 드라이버 호환

### 문제
Driver com.mysql.cj.jdbc.Driver claims to not accept jdbcUrl, jdbc:mariadb://...

### 원인
설치된 DB는 MariaDB인데 URL을 jdbc:mariadb://로 설정해서 충돌.

### 해결
MySQL 드라이버 + jdbc:mysql:// URL로 통일.
MariaDB는 MySQL과 호환되지만 URL 프로토콜은 맞춰야 함.

---

## 3. VO 오타 디버깅

### 문제
There is no getter for property named 'description' in DepartmentVO

### 원인
필드명 오타: describtion → description
타입 오류: createdAt을 String으로 선언 → LocalDateTime으로 수정

### 배운 점
MyBatis는 VO의 getter를 통해 파라미터를 바인딩함.
필드명 오타 하나로 전체 쿼리가 실패함.

---

## 4. 패키지 구조

com.hospital.reservation
├── controller  (REST API 엔드포인트)
├── service     (비즈니스 로직)
├── dao         (MyBatis 매퍼 인터페이스)
└── vo          (테이블 매핑 객체)

src/main/resources/mapper/ (SQL XML 파일)

---

## 5. Lombok 어노테이션 정리

@Getter / @Setter      → 읽기/쓰기 메서드
@NoArgsConstructor     → MyBatis 객체 생성용 기본 생성자
@AllArgsConstructor    → Builder 동작에 필요
@Builder               → 필요한 필드만 골라서 객체 생성

---

## 6. DEFAULT NOW() vs AUTO_INCREMENT

DEFAULT NOW()    → INSERT 시 자동으로 현재 시간 입력
AUTO_INCREMENT   → INSERT 시 자동으로 숫자 증가 (PK용)
둘 다 INSERT할 때 값 안 넣어도 DB가 자동으로 채워줌.

---

## 내일 할 것
- Postman으로 나머지 테이블 전체 테스트
- application.properties 비밀번호 .gitignore 처리
