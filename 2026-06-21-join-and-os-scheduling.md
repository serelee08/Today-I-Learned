# 2026-06-21 — 예약 조회 JOIN (정규화 설계) & OS 스케줄링

## 오늘 한 것
1. 예약 조회 시 의사명·진료과명을 함께 반환하도록 **JOIN 쿼리** 작성 (정규화 설계 활용)
2. 예약 폼 민트 테마 디자인 마무리
3. 정보처리기능사 운영체제 — **프로세스 스케줄링(디스패치)** 학습

---

## 1. 왜 JOIN이 필요한가 (정규화의 결과)

`appointment` 테이블엔 `doctor_no`, `patient_no` 같은 **번호(FK)**만 있다. 의사 이름·진료과명은 각각 `doctor`, `department` 테이블에 따로 있다.

- 정규화: 의사 정보는 `doctor`에 한 번만 저장, 예약은 `doctor_no`로 참조만
- 장점: 중복 제거, 데이터 무결성, 수정 용이 (의사 이름 바뀌면 한 줄만 고침)
- 대가: 조회 시 흩어진 데이터를 JOIN으로 합쳐야 함

→ 화면에 `의사 2`(번호)가 아니라 `박내과(내과)`(이름)를 보여주려면 JOIN 필수.

---

## 2. JOIN 쿼리 (appointment → doctor → department)

```sql
SELECT
    a.appointment_no, a.patient_no, a.doctor_no,
    a.appointment_dt, a.status, a.memo, a.created_at,
    p.name AS patient_name,
    d.name AS doctor_name,
    dep.department_name AS department_name
FROM appointment a
JOIN patient p      ON a.patient_no = p.patient_no
JOIN doctor d       ON a.doctor_no = d.doctor_no
JOIN department dep ON d.department_no = dep.department_no
ORDER BY a.appointment_no
```

핵심:
- `SELECT *` 대신 컬럼 명시 (production 기본 — 불필요한 컬럼 안 읽음)
- 테이블 별칭(a, p, d, dep)으로 가독성 확보
- **진료과는 두 단계 참조**: appointment엔 진료과가 직접 없음 → appointment → doctor → department로 타고 감 (정규화 설계의 결과)
- 한 번의 JOIN으로 다 가져와 **N+1 문제 회피**

---

## 3. JOIN만으로는 부족 — VO에 담을 그릇 필요

JOIN으로 가져와도 VO에 필드가 없으면 버려진다. AppointmentVO에 표시용 필드 추가:

```java
private String patientName;
private String doctorName;
private String departmentName;
```

`map-underscore-to-camel-case=true` 덕에 `patient_name` → `patientName` 자동 매핑.

---

## 4. 디버깅 기록 (오늘 실제로 겪음)

**증상:** JOIN 쿼리는 맞는데 JSON에 이름이 안 나옴
**가설 좁히기:**
- Mapper findAll에 JOIN 있나? → 있음 (확인)
- status·memo 한글이 깨져 보임 → 브라우저 표시 문제일 뿐, DB는 정상 (Postman에서 정상 확인)
- 그럼 원인은? → VO에 표시용 필드(patientName 등)가 없어서 담을 곳이 없음
**교훈:**
- "깨져 보인다"와 "실제로 깨졌다"는 다르다 — 증거(Postman)로 검증
- 구조를 알려면 데이터 한 줄이 아니라 **스키마(ERD)**를 봐라
- JOIN이 데이터를 가져와도 **받을 객체에 필드가 없으면 버려진다**

---

## 5. 발견한 설계 개선점 (면접 답변 자산)

ERD를 보며 스스로 발견:
- `doctor` 테이블에 `specialty`(텍스트)와 `department_no`(FK)가 **둘 다** 존재 → 진료과 정보 중복 (부분 비정규화)
- 개선: `department_no` FK로 단일화, `specialty` 제거 → JOIN으로 진료과명 조회
- `AppointmentVO`에 DB 필드와 표시 필드가 섞임 → Entity / DTO 분리가 정석 (단일 책임)

→ 면접에서 "DB 설계 아쉬운 점?" 질문에 답할 수 있는 재료.

---

## 6. 운영체제 — 프로세스 스케줄링 (정보처리기능사)

**스케줄링**: CPU를 어떤 프로세스에 먼저 줄지 정하는 것
**디스패치(Dispatch)**: 준비(ready) 상태 프로세스를 실제 CPU(running)에 올리는 동작

**비선점(Non-preemptive)** — 한번 CPU 잡으면 끝날 때까지 안 뺏음
- FCFS (First Come First Served): 먼저 온 순서대로. 단점 — 긴 작업이 앞에 오면 뒤가 다 기다림(콘보이 효과)
- SJF (Shortest Job First): 실행시간 짧은 것 먼저. 평균 대기시간 최소. 단점 — 긴 작업 기아(starvation)
- 우선순위(Priority): 우선순위 높은 것 먼저
- HRN: SJF의 기아 보완. 우선순위 = (대기시간 + 실행시간) / 실행시간

**선점(Preemptive)** — 더 급한 게 오면 CPU 뺏음
- SRT (Shortest Remaining Time): SJF의 선점 버전
- Round Robin: 시간 할당량(time quantum)만큼만 쓰고 교체. 시분할 시스템 기본
- 다단계 큐(Multi-level Queue)

**개발자 관점 연결:** 디스패치는 OS가 자원을 분배하는 방식. DB의 트랜잭션 락/스케줄링, 서버의 요청 처리 순서도 같은 원리(자원을 누구에게 먼저)다. 운영체제 개념이 실제 백엔드/DB 동작 이해의 바탕이 됨.

---

## 회고
오늘 핵심은 "정규화 설계가 왜 JOIN을 부르는가"를 이해한 것. 그리고 ERD를 읽으며 내 설계의 중복(specialty vs department_no)을 스스로 발견한 것. 디버깅에선 "깨져 보임 ≠ 깨짐"을 증거로 검증하는 훈련을 했다. 다음: VO 필드 추가 반영 확인 → 프론트에서 이름 표시 → 예약 조회 화면.

## 다음 단계 (TODO)
- [ ] AppointmentVO 표시 필드 반영 후 JSON에서 이름 확인
- [ ] 프론트 예약 목록에 의사명·진료과명 표시
- [ ] AppointmentVO → Response DTO 분리 (Entity/DTO 분리)
- [ ] doctor.specialty 중복 제거 → department_no FK로 단일화
