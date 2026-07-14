# 2026-07-14 — D365 F&SCM: PO Change Management 활성화 & Workflow 승인자 디버깅 (전체 경로 기록)

## 오늘 한 일

**2.3 Process purchase orders** 학습 후, USMF 데모 환경에서 **Change management를 실제로 켜고, 워크플로우 승인자를 설정하고, PO를 Submit해서 승인 절차가 실제로 작동하는지** 끝까지 검증했다. 중간에 승인자가 의도한 사람과 다르게 배정되는 문제를 만나서 원인을 찾고 해결까지 완료.

---

## 전체 작업 경로 (실제로 클릭한 순서 그대로)

### 1단계 — Change Management 활성화

```
Procurement and sourcing > Setup > Procurement and sourcing parameters
→ General 탭
→ "Change management for purchase orders" 섹션
→ Activate change management 체크
```

**의미:** 이제부터 PO는 승인 워크플로우를 거쳐야만 Confirm 가능하도록 규칙을 켠 것.

### 2단계 — 기존 워크플로우 존재 확인

```
Procurement and sourcing > Setup > Procurement and sourcing workflows
→ 목록에서 "Purchase order workflow" (ID: 000063) 확인
→ Association: usmf, Type: PurchTableTemplate
```

**발견:** 이 워크플로우는 2017년에 이미 Microsoft가 만들어놓은 것이었고, Active 상태였다. 새로 만들 필요 없이 이미 존재했음.

### 3단계 — 워크플로우 편집기(Workflow Editor) 실행

```
Purchase order workflow > Versions > 해당 버전 선택 > Edit
```

- Workflow Editor는 브라우저 안이 아니라 **별도 실행 프로그램(ClickOnce)**으로 열림
- 최초 실행 시 "게시자를 확인할 수 없습니다" 보안 경고 뜸 → 정상, 실행(Run) 승인하면 됨

### 4단계 — 승인자(Approver) 확인 및 수정

```
Workflow Editor 안에서
→ "Approve purchase order" 박스 클릭 (선택만, 더블클릭으로 Properties 안 열림)
→ 상단 툴바의 "Properties" 버튼 클릭 → 우측에 Properties 패널 표시
→ Assignment type 필드 확인
```

- Assignment type을 **User**로 설정
- User 필드에 본인 계정(serena.lee) 추가

### 5단계 — 저장 및 활성화

```
Save 클릭
→ "새 버전을 저장하고 활성화하시겠습니까?" 팝업
→ Activate 선택
→ Editor 닫기(Close)
```

### 6단계 — 실제 PO에 Submit 테스트

```
Procurement and sourcing > Purchase orders > All purchase orders
→ PO 00000380 (Vendor 1001, Acme Office Supplies) 열기
→ Purchase 탭 > Workflow 그룹 > Submit 클릭
→ 코멘트 입력 후 확인
```

**결과:** PO 상태가 `Open order / Draft` → `Open order / In review`로 변경됨. 워크플로우 자체는 정상 작동 확인.

### 7단계 — 승인자 확인 (Workflow history)

```
PO 화면 상단 > "Workflow history" 링크 클릭
→ Tracking details list 확인
```

여기서 문제를 발견:
```
Work item | Creation | "Assigned to user: Inga Numadutir"
```

**본인이 아니라 "Inga Numadutir"라는 다른 계정에게 승인 요청이 가 있었음.**

### 8단계 — 원인 파악

Workflow Editor로 다시 들어가서 Assignment 설정의 User 필드를 재확인한 결과:
```
User 필드 목록에 serena.lee와 Inga Numadutir가 둘 다 들어있었음
→ 시스템이 그중 Inga를 먼저 선택해서 배정한 것
```

### 9단계 — 기존 PO Recall (승인 요청 취소)

```
PO 00000380 화면 (Workflow history 화면 상단)
→ "Recall" 클릭
```
- PO 상태가 `In review` → `Draft`로 되돌아감
- **주의:** Recall은 그 PO 하나의 승인 요청만 취소하는 것이고, 워크플로우 자체(000063)나 다른 PO에는 영향 없음

### 10단계 — Assignment에서 Inga 제거

```
Workflow Editor > Approve purchase order > Properties > Assignment type = User
→ User 필드 목록에서 "Inga Numadutir" 선택 → Remove
→ serena.lee만 남은 것 확인
→ Save → Activate
```

### 11단계 — 불필요한 워크플로우 버전 정리 (Inactive 버전 삭제)

```
Workflow versions 화면에 총 3개 버전 존재
→ Active 아닌 버전 2개만 선택하여 Delete
→ Active 버전 1개만 남기고 정리
```
**주의:** Active 버전은 절대 삭제하면 안 됨. 삭제 전 반드시 진행 중인 PO를 먼저 Recall 해서 정리해야 안전.

### 12단계 (다음에 이어서 할 것) — 재검증

```
PO 00000380 다시 열기 (Draft 상태) → Submit
→ 우측 상단 알림(🔔) 또는 System administration > Work items에서
→ 이번엔 본인에게만 승인 요청이 오는지 최종 확인
```

---

## 오늘 배운 개념 (2.3 모듈 핵심)

### Change Management — 승인 절차 강제 스위치

```
OFF: PO 작성 완료 → 바로 Confirm 가능
ON:  PO 작성 완료 → Submit → 워크플로우 승인 → Approved → Confirm 가능
```

**공식 문서 핵심 문장:**
> "Change management must be activated **and** a workflow must be set up for the Submit button to appear and be functional."

→ 파라미터 활성화 + 워크플로우 존재(Active) 둘 다 있어야 Submit 버튼이 나타난다.

### PO 승인 상태 5단계

| 상태 | 의미 | 재고 트랜잭션 |
|---|---|---|
| Draft | Submit 전 | 없음 |
| In review | Submit 직후, 워크플로우 진행 중 | 없음 |
| Approved | 워크플로우 승인 완료 | 있음 (재고품목) |
| Confirmed | 벤더에게 공식 전달 | 있음 |
| Finalized | 재무적으로 완전 종결 | 있음 |

### Change Management 역할 4가지

| 역할 | 하는 일 |
|---|---|
| Requester | PO 제출, 변경 요청 생성/제출, Recall |
| Approver | Approve / Edit and Approve / Reject / Request Changes |
| Task user | 검토 작업 수행 (선택) |
| Administrator | 워크플로우 설계 |

### Vendor Returns — Credit note로 인보이스 역방향 복사

```
Procurement and sourcing > Purchase orders > All purchase orders
→ New → Vendor 지정 → Action Pane > Purchase > Credit note
→ 기존 Vendor Invoice 라인 복사
```
자동 적용 옵션: Invert sign(부호 반전) / Copy charges(부대비용 복사) / Copy precisely(원본 그대로 복사)

---

## 막혔던 점 & 해결 (오늘의 진짜 핵심)

| 막힌 점 | 원인 | 해결 |
|---|---|---|
| PO 화면에 Submit 버튼이 안 보임 | Change management 파라미터만 켜고, 워크플로우 자체 확인 안 함 | 워크플로우가 이미 존재/Active임을 확인 → 파라미터+워크플로우 둘 다 필요하다는 것 이해 |
| Workflow Editor에서 더블클릭해도 Properties 안 열림 | 클릭 방식 문제 | 박스를 한 번 클릭(선택) 후 상단 툴바의 "Properties" 버튼으로 우측 패널 여는 방식으로 대체 |
| Submit 후 본인 알림에 승인 요청이 안 뜸 | 승인 요청이 본인이 아니라 "Inga Numadutir"에게 감 | Workflow history의 Tracking details list에서 실제 배정 대상 확인 → 원인 추적 |
| Assignment에 본인 이름을 넣었는데도 다른 사람에게 감 | User 필드에 기존 값(Inga)이 삭제 안 되고 본인 이름과 함께 남아있었음 | User 필드 목록에서 Inga를 선택해 Remove, 본인만 남기고 재저장/재활성화 |
| 이미 제출된 PO(00000380)에 워크플로우 수정이 반영 안 됨 | Submit 시점의 워크플로우 버전이 그 PO에 고정되고, 이후 수정은 소급 적용 안 됨 | **Recall**로 그 PO의 진행 중인 승인 요청을 취소하고 Draft로 되돌린 뒤, 수정된 워크플로우로 재제출해야 함 |
| Workflow versions에 버전이 3개 쌓여서 정리하려 함 | 연습 중 여러 번 Save/Activate 반복 | Active 버전은 유지, 나머지 Inactive 2개만 Delete. **단, 진행 중인 PO를 먼저 Recall 해서 정리한 뒤에 삭제해야 안전** (Active 아닌 버전이라도 진행 중인 PO가 참조 중일 수 있음) |

---

## 핵심 교훈 — Recall vs Delete 구분

```
Recall (PO 단위) = 특정 PO 하나의 진행 중인 승인 요청만 취소. 안전함.
Delete (Workflow Versions 단위) = 회사 전체의 결재 시스템 버전 자체를 삭제. 
                                   Active 버전을 지우면 전사 PO 승인 절차가 깨짐. 위험함.
```

이 둘을 구분 못 하면 워크플로우 자체를 잘못 건드릴 뻔했던 게 오늘의 가장 큰 리스크 포인트였다.

---

## 더 공부할 것

- [ ] 재제출한 PO 00000380이 이번엔 본인에게만 승인 요청이 오는지 최종 확인 (다음 세션에서 이어서)
- [ ] Approve 버튼 눌러서 실제 승인까지 완료 → PO 상태가 Approved로 바뀌는지 확인
- [ ] Assignment type의 다른 옵션(Participant, Requester's manager 등) 차이와 실무에서 어떻게 쓰이는지
- [ ] Compare purchase order versions — 변경 이력 비교 화면
- [ ] Vendor return을 Credit note로 실제로 끝까지 생성해보기

---

**Author:** [@serelee08](https://github.com/serelee08)
