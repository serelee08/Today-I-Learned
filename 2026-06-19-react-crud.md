# 2026-06-19 — 예약 CRUD 완성 (POST / PATCH / DELETE + 검증)

## 오늘 한 것
React 예약 시스템에 등록·상태변경·취소 기능을 붙여 예약 CRUD를 거의 완성했다. HTTP 메서드 4종(GET/POST/PATCH/DELETE)을 모두 사용.

## 1. 오늘 완성한 흐름
- GET 예약 목록 조회 (이미 있던 것)
- POST 예약 등록 + DB 저장
- PATCH 예약 상태 변경 (예약→완료→취소)
- DELETE 예약 삭제
- 입력 검증(validation)

## 2. 예약 등록 (POST) — 날짜/시간 분리
- 시간 옵션 자동 생성 (for문, 09:00~17:30 30분 단위)
- padStart(2,'0') = 9를 "09"로
- 전송 시 합치기: appointmentDate + 'T' + appointmentTime + ':00'
- = "2026-06-20T10:30:00" (백엔드 LocalDateTime 형식)

## 3. 입력 검증 (Validation)
- handleSubmit 맨 앞에서 빈 값 검사
- !값 = "비어있으면", return = 함수 즉시 멈춤 → 저장 안 됨
- 잘못된 입력을 미리 막는 것이 좋은 개발의 기본

## 4. 예약 상태 변경 (PATCH)
- PATCH = 일부만 수정 (상태값만)
- 주소에 ?status=완료 형식으로 전달 (@RequestParam)
- 상태취소(기록 보존) vs 삭제(DB 제거)는 다름

## 5. 예약 삭제 (DELETE)
- DELETE = 삭제, body 없이 주소에 번호만
- 삭제 전 window.confirm() 확인창
- 삭제 후 목록 갱신

## 6. HTTP 메서드 정리
- GET 조회 / POST 생성 / PATCH 일부수정 / DELETE 삭제
- 같은 주소라도 메서드에 따라 동작 다름
- 백엔드 Controller = API 제공, React api.js = API 호출

## 7. 부모-자식 소통 (props)
- App에서 onSuccess={loadAppointments} 전달
- ReservationForm({ onSuccess })에서 받아 호출
- 자식이 부모에게 "끝났어"를 알리는 패턴

## 회고
하루 만에 예약 CRUD 한 세트와 검증까지 완성. HTTP 메서드 4종을 직접 써보며 REST API가 손에 익었다. "빈 값 막기"를 스스로 떠올린 게 의미 있었다. 다음은 예약 수정(PUT), 컴포넌트 분리, 디자인.
git commit -m "docs: add 예약 CRUD TIL (2026-06-19)"
git push
