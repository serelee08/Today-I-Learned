# 2026-07-14 — D365 F&SCM: Process Purchase Orders & Change Management Workflow

## 오늘 한 일

**2.3 Process purchase orders in Dynamics 365 Supply Chain Management** 학습
(SCF Ch.4 Purchase Orders / PROC Ch.7 Purchase Orders / DAT Ch.6, PROC Ch.8 Vendor Returns 대응)

개념 학습 후, USMF 데모 환경에서 **PO Change Management를 실제로 켜고, 워크플로우를 만들고 Activate까지** 직접 진행했다.

---

## 오늘 배운 것 (기술적으로)

### 1. PO Change Management — 승인 절차 강제 스위치

Change management를 켜면 PO가 곧바로 Confirm 되지 않고, **승인 워크플로우를 거쳐야만 Confirm 가능**해진다.

```
Change management OFF: PO 작성 완료 → 바로 Confirm 가능
Change management ON:  PO 작성 완료 → Submit → 승인 워크플로우 → Approved → Confirm 가능
```

**승인 상태(Approval status) 5단계:**

| 상태 | 의미 | 재고 트랜잭션 발생 여부 |
|---|---|---|
| Draft | Submit 누르기 전 | 없음 |
| In review | Submit 누른 직후, 워크플로우 진행 중 | 없음 |
| Approved | 워크플로우 승인 완료 | 있음 (재고품목 라인에 한해) |
| Confirmed | PO Confirm 완료, 벤더에게 공식 전달 | 있음 |
| Finalized | 재무적으로 완전히 종결, 더 이상 변경 불가 | 있음 |

**Change management 활성화 경로:**
```
Procurement and sourcing > Setup > Procurement and sourcing parameters
→ General 탭 > "Change management for purchase orders" 섹션
→ Activate change management 체크
→ (선택) Allow override of settings per vendor 체크
  → 체크하면 Vendor 페이지의 Purchase order defaults 탭에서 벤더별로 개별 설정 가능
```

**공식 문서 핵심 문장:**
> "Change management must be activated **and** a workflow must be set up for the Submit button to appear and be functional."

→ 파라미터만 켜는 걸로는 부족하고, **워크플로우 자체가 별도로 만들어져서 Active 상태여야** Submit 버튼이 실제로 나타난다. 이 두 조건이 오늘 헤맸던 핵심 원인이었다.

### 2. Change Management의 4가지 역할

| 역할 | 하는 일 |
|---|---|
| Requester | PO 최초 제출, 변경 요청 생성/제출, Recall(회수) |
| Approver | Approve / Edit and Approve / Reject / Request Changes |
| Task user (선택) | 검토 작업 수행 |
| Administrator | 워크플로우 자체를 설계 |

### 3. Workflow Editor — 별도 프로그램(ClickOnce)으로 열림

워크플로우 편집은 브라우저 안에서가 아니라 **별도로 다운로드/실행되는 프로그램**에서 이루어진다. 처음 열 때 "게시자를 확인할 수 없습니다" 보안 경고가 뜨는데, 이건 정상이며 실행(Run) 승인하면 된다.

**워크플로우 구조 확인/수정 절차:**
```
Procurement and sourcing > Setup > Procurement and sourcing workflows
→ "Purchase order workflow" (000063) 선택 → Versions
→ 버전 선택 → Edit → Workflow Editor 실행
→ "Approve purchase order" 박스 클릭 (선택만, 더블클릭 안 먹힘)
→ 상단 바 "Properties" 클릭 → 우측에 Properties 패널 표시
→ Assignment type 필드 확인/수정
```

**Assignment type**이 승인자를 결정하는 핵심 필드. User(특정 계정 직접 지정) 또는 Participant(역할/조건 기반) 중 선택 가능. 오늘은 User 타입으로 본인 계정을 직접 지정해봤다.

### 4. Vendor Returns — Credit note로 인보이스 역방향 복사

반품은 새 PO를 처음부터 만드는 게 아니라, 기존 Vendor Invoice를 **Credit note** 기능으로 복사해서 만든다.

```
Procurement and sourcing > Purchase orders > All purchase orders
→ New (반품용 PO) → Vendor account 지정
→ Action Pane > Purchase > Credit note
→ 기존 Vendor Invoice 선택해서 라인 복사
```

자동 적용되는 3가지 잠금 옵션:
- **Invert sign**: 수량/금액 부호 반전 (+100 → -100)
- **Copy charges**: 부대비용도 상쇄되도록 복사
- **Copy precisely**: 원본 인보이스 필드값을 정확히 그대로 복사

---

## 막혔던 점 & 해결

| 막힌 점 | 원인 | 해결 |
|---|---|---|
| PO 화면에 Workflow/Submit 버튼이 안 보임 | Change management 파라미터만 켜고, 워크플로우 자체는 아직 미확인 상태였음 | 워크플로우가 이미 2017년에 만들어져 Active 상태인 걸 확인. 파라미터 활성화 + 워크플로우 Active 둘 다 필요하다는 것 이해 |
| Workflow Editor에서 Properties 창이 더블클릭으로 안 뜸 | 클릭 방식/타이밍 문제로 추정 | 박스를 한 번 클릭(선택)한 후, 상단 툴바의 "Properties" 버튼으로 우측 패널을 여는 방식으로 대체 |
| Submit 눌러도 본인 알림에 승인 요청이 안 뜸 | 본인은 Requester 역할이지 Approver가 아니었기 때문 (정상 동작) | Assignment type을 확인해, Approver가 다른 역할/계정으로 지정되어 있으면 본인 알림에 안 뜨는 게 맞다는 것을 이해. 테스트를 위해 Assignment type을 User로 바꾸고 본인 계정을 직접 지정 |

---

## 더 공부할 것 (솔직한 메모)

- [ ] 워크플로우 수정 후 재저장 → 재활성화(Activate)까지 실제로 완료해서, PO Submit → 본인 알림에 승인 요청이 뜨는지 끝까지 확인
- [ ] Participant 기반 Assignment (역할/조건별 자동 배정)의 구체적 설정 방법
- [ ] Compare purchase order versions — 변경 이력 비교 화면 사용법
- [ ] Item arrivals / Arrival overview 워크스페이스 실습
- [ ] Vendor return을 Credit note로 실제로 처음부터 끝까지 생성해보기 (오늘은 개념만 정리, 실습은 아직)

---

**Author:** [@serelee08](https://github.com/serelee08)
