# 2026-06-18 — 예약 등록 (POST) & REST API 연동

## 오늘 한 것
React 예약 폼의 [예약] 버튼을 백엔드와 연동해, 입력한 예약이 **실제로 DB에 저장**되도록 구현했다. (POST 연동 완성)

---

## 1. 풀스택 데이터 흐름 (오늘 완성한 것)

```
React 입력 → fetch(POST) → Spring Boot Controller → Service → DAO → MySQL 저장
```

지금까지는 데이터 "읽기(GET)"만 했지만, 오늘 데이터 "쓰기(POST)"까지 완성. 예약 시스템의 핵심 기능이 작동.

---

## 2. POST 요청 (api.js)

```javascript
export async function createAppointment(appointment) {
  const res = await fetch(`${BASE_URL}/appointments`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(appointment),
  })
  return res.json()
}
```

- `method: 'POST'` = 가져오기(GET) 아닌 보내기(POST)
- `headers` = "JSON 형식으로 보낼게" 명시
- `body: JSON.stringify(...)` = 객체를 JSON 문자열로 변환해 전송

---

## 3. handleSubmit — 실제 전송

```jsx
const handleSubmit = async () => {
  const appointment = {
    patientNo: 1,
    doctorNo: Number(doctorNo),
    appointmentDt: appointmentDate,
    status: '예약',
    memo: patientName + ' 예약',
  }
  try {
    const result = await createAppointment(appointment)
    alert('예약이 완료되었습니다!')
  } catch (err) {
    alert('예약에 실패했습니다.')
  }
}
```

---

## 4. 오늘 겪은 핵심 포인트

**① 컬럼명 정확히 맞추기**
- 백엔드 VO는 `appointmentDate`가 아니라 `appointmentDt`
- 프론트에서 보내는 키 이름이 백엔드와 다르면 null로 들어감

**② 타입 변환**
- 드롭다운에서 고른 값은 문자열("1") → `Number(doctorNo)`로 숫자 변환
- 백엔드가 Long(숫자)을 기대하기 때문

**③ handleSubmit 교체 누락 주의**
- 테스트용 코드(`alert('전송 전')`)가 남아있어서 한참 헷갈림
- 코드를 바꿨으면 저장 + 실제 반영 확인 필수

---

## 5. REST API 개념 정리

REST API = 백엔드가 "주소 + HTTP 메서드"로 데이터를 다루는 규칙.

| 메서드 | 의미 | 예시 |
|---|---|---|
| GET | 조회 | `/api/appointments` 목록 |
| POST | 생성 | `/api/appointments` 등록 |
| PUT | 수정 | `/api/appointments/{no}` |
| DELETE | 삭제 | `/api/appointments/{no}` |

- 같은 주소라도 메서드에 따라 동작이 다름
- **API 호출** = 백엔드에게 "이거 해줘"라고 요청하는 것 (fetch가 그 역할)
- 백엔드 Controller = REST API "제공", React api.js = REST API "호출"

---

## 6. DB 확인 (DBeaver)

```sql
USE hospital_reservation;
SELECT * FROM appointment ORDER BY appointment_no DESC;
```

- React에서 등록한 예약이 appointment 테이블에 새 행으로 저장된 것 확인
- 화면 → 백엔드 → DB까지 데이터가 끝까지 흘러간 것을 눈으로 검증

---

## 회고
오늘 "쓰기(POST)" 전체 흐름을 완성했다. React에서 입력한 데이터가 REST API를 거쳐 DB까지 저장되는 풀스택 한 사이클을 직접 구현. 컬럼명 불일치, 타입 변환 같은 실무에서 자주 겪는 버그도 경험했다. 다음은 예약 후 목록 자동 갱신, 환자 정보 실제 저장.
