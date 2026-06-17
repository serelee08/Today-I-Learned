# 2026-06-17 — React 컴포넌트 & 입력 폼 (예약하기)

## 오늘 한 것
병원 예약 시스템 React 프론트엔드에 **예약하기 폼(ReservationForm)** 컴포넌트를 만들고 App.jsx에 연결했다.

---

## 1. 컴포넌트 분리

화면을 조각(컴포넌트)으로 나눠서 만들고, 필요한 곳에서 가져다 쓰는 구조.

```
src/
├── api/api.js              # 백엔드 호출 함수 모음
├── components/
│   └── ReservationForm.jsx # 예약 입력 폼 (새로 만듦)
└── App.jsx                 # 컴포넌트 조립
```

- 컴포넌트 이름은 첫 글자 대문자 (React 규칙)
- `export default`로 내보내고, App.jsx에서 `import`로 가져옴
- 사용: `<ReservationForm />`

---

## 2. import 경로

```jsx
// components 폴더 안에서 api 폴더의 파일을 가져올 때
import { getDepartments, getDoctors } from '../api/api'
```

- `..` = 한 단계 위 폴더로 이동
- 같은 폴더면 `./`, 위 폴더면 `../`
- 경로 틀리면 함수를 못 찾아 데이터가 안 들어옴 (오늘 겪은 버그)

---

## 3. 입력값 관리 (useState)

입력값은 화면이 아니라 **별도의 상태(상자)**에 저장한다.

```jsx
const [patientName, setPatientName] = useState('')
```

- `patientName` = 값을 읽는 상자
- `setPatientName` = 값을 저장하는 함수
- 처음 값: 글자면 `''`, 숫자면 `0`, 목록이면 `[]`

---

## 4. onChange — 입력값을 상태에 저장

```jsx
<input value={patientName} onChange={(e) => setPatientName(e.target.value)} />
```

- `onChange` = 값이 바뀔 때마다 실행
- `e.target.value` = 방금 입력/선택된 값
- `value` = 상자 값을 화면에 표시 (보여주기)
- `onChange` = 입력값을 상자에 저장 (저장하기)
- 둘은 짝꿍 — 함께 있어야 입력칸이 정상 작동

**입력 종류별:**
- input(타이핑), select(드롭다운 선택) → `onChange`
- button(클릭) → `onClick`

---

## 5. 드롭다운 (select + map)

```jsx
<select value={departmentNo} onChange={(e) => setDepartmentNo(e.target.value)}>
  <option value="">선택하세요</option>
  {departments.map((d) => (
    <option key={d.departmentNo} value={d.departmentNo}>
      {d.departmentName}
    </option>
  ))}
</select>
```

- `map` = 배열을 하나씩 돌며 화면 요소로 변환 (5개면 5번 반복)
- `d` = 반복 중 현재 항목 (이름 자유)
- `key` = React가 항목 구분하는 값 (목록 만들 때 필수)
- `onChange`는 `<select>`에 붙음 (`<option>`엔 없음)

---

## 6. 오늘의 버그 & 해결

**문제:** 드롭다운에 항목이 안 나오고 "선택하세요"만 표시됨
**원인:** ReservationForm.jsx에서 `api.js` import가 빠짐
**해결:** `import { getDepartments, getDoctors } from '../api/api'` 추가

→ 콘솔에 에러가 없어도 데이터가 안 나올 수 있다. import 누락은 조용히 실패한다.

---

## 회고
화면을 컴포넌트로 분리하면 코드가 깔끔해지고 재사용이 쉬워진다. import 경로(`..`)와 onChange-useState 흐름이 핵심. 다음은 [예약] 버튼을 실제 백엔드로 보내는 POST 연동.
