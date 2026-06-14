# 2026-06-13 TIL — SQL: DDL / DML / DCL

## SQL 분류 3가지

SQL은 목적에 따라 3가지로 나뉨.

| 분류 | 이름 | 역할 |
|------|------|------|
| DDL | 데이터 정의어 | 테이블 구조 만들고 변경 |
| DML | 데이터 조작어 | 데이터 조회/삽입/수정/삭제 |
| DCL | 데이터 제어어 | 권한 부여/취소, 트랜잭션 제어 |

---

## DDL (Data Definition Language) — 데이터 정의어

**역할:** 테이블, 인덱스 등 DB 구조를 만들고 변경하는 명령어

```sql
CREATE  -- 테이블 생성
ALTER   -- 테이블 구조 변경
DROP    -- 테이블 삭제
TRUNCATE -- 테이블 데이터 전체 삭제 (구조는 유지)
```

**예시:**
```sql
-- 테이블 생성
CREATE TABLE department (
    department_no BIGINT PRIMARY KEY AUTO_INCREMENT,
    department_name VARCHAR(100) NOT NULL
);

-- 컬럼 추가
ALTER TABLE department ADD COLUMN phone VARCHAR(20);

-- 테이블 삭제
DROP TABLE department;
```

**특징:**
- 자동 COMMIT → 실행하면 되돌릴 수 없음
- 테이블 구조 자체를 바꾸는 명령어

---

## DML (Data Manipulation Language) — 데이터 조작어

**역할:** 테이블 안의 데이터를 다루는 명령어

```sql
SELECT  -- 데이터 조회
INSERT  -- 데이터 삽입
UPDATE  -- 데이터 수정
DELETE  -- 데이터 삭제
```

**예시:**
```sql
-- 조회
SELECT * FROM department;

-- 삽입
INSERT INTO department (department_name) VALUES ('내과');

-- 수정
UPDATE department SET department_name = '외과' WHERE department_no = 1;

-- 삭제
DELETE FROM department WHERE department_no = 1;
```

**특징:**
- COMMIT 전에 ROLLBACK 가능
- 실제 데이터를 조작하는 명령어

---

## DCL (Data Control Language) — 데이터 제어어

**역할:** 사용자 권한 관리 및 트랜잭션 제어

```sql
GRANT    -- 권한 부여
REVOKE   -- 권한 취소
COMMIT   -- 트랜잭션 확정 (저장)
ROLLBACK -- 트랜잭션 취소 (되돌리기)
```

**예시:**
```sql
-- 권한 부여
GRANT SELECT ON department TO user1;

-- 권한 취소
REVOKE SELECT ON department FROM user1;

-- 트랜잭션 확정
COMMIT;

-- 트랜잭션 취소
ROLLBACK;
```

**특징:**
- COMMIT: 변경사항 영구 저장
- ROLLBACK: 마지막 COMMIT 이후 변경사항 취소

---

## 핵심 암기

```
DDL → CREATE, ALTER, DROP, TRUNCATE (구조)
DML → SELECT, INSERT, UPDATE, DELETE (데이터)
DCL → GRANT, REVOKE, COMMIT, ROLLBACK (권한/제어)
```

---

## 실무 연결

```
hospital_reservation 프로젝트에서:

DDL → 테이블 만들 때 (CREATE TABLE department...)
DML → CRUD API 만들 때 (SELECT, INSERT, UPDATE, DELETE)
DCL → 트랜잭션 처리할 때 (@Transactional → COMMIT/ROLLBACK)
```
