# TIL - 2026.06.15

## 오늘 한 것
Hospital Reservation System 테스트 데이터 적재 + FK 제약조건 디버깅

---

## 1. FK 제약조건 위반 (Foreign Key Constraint)

### 문제
appointment 데이터 INSERT 시 에러:
Cannot add or update a child row: a foreign key constraint fails
appointment FOREIGN KEY (patient_no) REFERENCES patient(patient_no)

### 원인
appointment에 patient_no = 2를 넣으려 했으나, patient 테이블에 2번이 없었음.
DELETE 후 AUTO_INCREMENT를 리셋하지 않아 번호가 1, 3, 4, 5, 6으로 들어감.

### 해결
전체 테이블 삭제 후 AUTO_INCREMENT 초기화:
DELETE FROM medical_record;
DELETE FROM appointment;
DELETE FROM doctor;
DELETE FROM patient;
DELETE FROM department;
ALTER TABLE department AUTO_INCREMENT = 1;
(나머지 테이블도 동일)

### 배운 점
FK 자식 테이블은 부모 테이블에 존재하는 값만 참조 가능.
INSERT 순서: department → patient → doctor → appointment → medical_record

---

## 2. AUTO_INCREMENT 동작 원리

### 핵심
- DELETE해도 AUTO_INCREMENT 카운터는 되돌아가지 않음
- 중간 번호 삭제 시 그 번호는 재사용 안 됨 (1 다음 3으로 건너뜀)
- 번호를 1부터 다시 시작하려면 ALTER TABLE ... AUTO_INCREMENT = 1 필요

### 실무 포인트
PK 번호가 중간에 비어도 기능상 문제 없음.
PK는 순서가 아니라 고유성(uniqueness)이 중요.

---

## 3. UNIQUE 제약조건 위반

### 문제
Duplicate entry 'patient01' for key 'member_id'

### 원인
member_id는 UNIQUE 컬럼인데 patient01을 두 번 INSERT 시도.

### 배운 점
UNIQUE 제약은 중복 값을 막는다.
회원 ID, 이메일 등 고유해야 하는 컬럼에 사용.

---

## 4. FK INSERT 순서 정리

부모 테이블 먼저, 자식 테이블 나중:

department (독립)
   ↓
patient (독립) / doctor (department 참조)
   ↓
appointment (patient + doctor 참조)
   ↓
medical_record (appointment + patient + doctor 참조)

---

## 내일 할 것
- hospital README + API 명세 정리
