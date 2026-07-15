# D365 SCM 컨설턴트 학습 로드맵

> 3일차 기준 작성. 완벽한 이해가 목표가 아니라, 전체 흐름을 먼저 잡고 이후 실무에서 깊이를 채워가는 구조.

---

## 0. 학습 원칙 (먼저 읽기)

- **깊이보다 속도.** 한 모듈을 완벽히 이해하고 넘어가는 게 아니라, 전체를 1회독한 뒤 필요한 부분만 다시 깊게 판다.
- **화면 클릭이 아니라 "실제로 무슨 일이 일어나는지"로 이해한다.** 버튼 이름을 외우지 말고, 그 버튼이 현실에서 어떤 순간을 흉내내는지 생각한다.
- **작업 후 항상 "그럼 재고는? 회계는?"을 확인하는 습관을 들인다.** (Inventory transactions, Voucher transactions)
- **표준과 실무는 다를 수 있다는 것을 항상 의심한다.** 배운 대로 다 맞다고 가정하지 않는다.

---

## 1단계 — 마스터 데이터 (모든 것의 전제조건)

**개념:** Customer / Vendor / Item은 단순 이름표가 아니라, 각각 "그룹(Group)"을 통해 회계 계정과 자동 연결된다.

| 항목 | 경로 | 확인할 것 |
|---|---|---|
| Customer 등록 | Accounts receivable > Customers > All customers > New | Customer group → 나중에 매출채권(AR) 계정과 연결 |
| Vendor 등록 | Accounts payable > Vendors > All vendors > New | Vendor group → 나중에 매입채무(AP) 계정과 연결 |
| Item 등록 및 릴리즈 | Product information management > Products > New / Released products | Legal entity(USMF)에 릴리즈해야 실제 사용 가능 |

**체크:** 세 가지를 등록할 때, "그룹" 필드가 왜 있는지 설명할 수 있는가?

---

## 2단계 — 구매 사이클 (Procure-to-Pay)

**흐름:**
```
PO 생성 → Confirm → Product Receipt(입고) → Vendor Invoice → Payment → Settle
```

| 단계 | 경로 | 이 시점에 일어나는 일 |
|---|---|---|
| PO 생성/확정 | Procurement and sourcing > Purchase orders > New → Confirm | "주문서만 던진" 상태. 재고·회계 영향 없음 |
| Product Receipt | PO 화면 > Receive > Product receipt (Site/Warehouse/Location 지정) | InventTrans 발생, 재고 수량 증가. 회계는 잠정(accrual)만 |
| Vendor Invoice | Vendor invoice 처리 (3-way matching: PO-입고-인보이스 대사) | 매입채무(AP) 확정, 입고 시 잠정계정 정리 |
| Payment | Vendor payment journal / Payment proposal → Settle | 실제 자금 지급, AP 청산 |

**핵심 교훈:** 입고는 항상 송장보다 먼저 온다 (선급 예외 제외). 물리적 확정과 재무적 확정은 별개 단계.

**체크:** "Product Receipt와 Vendor Invoice의 차이"를 화면 없이 설명할 수 있는가?

---

## 3단계 — 판매 사이클 (Order-to-Cash)

**흐름:** 구매의 완전한 대칭 구조 (방향만 반대, 로직은 동일)
```
SO 생성 → Confirm → Packing slip(출고) → Sales Invoice → Customer Payment → Settle
```

| 단계 | 경로 | 이 시점에 일어나는 일 |
|---|---|---|
| SO 생성/확정 | Sales and marketing > Sales orders > New → Confirm | 수주 확정 |
| Packing slip | 출고 처리 (Warehouse/Location 지정) | InventTrans 반대방향 발생, 재고 감소 |
| Sales Invoice | 인보이스 발행 | 매출채권(AR) 확정 |
| Customer Payment | Customer payment journal → Settle | 수금 처리, AR 청산 |

**체크:** 구매와 판매, 어느 부분이 대칭이고 어느 부분이 다른지 짝지어 설명할 수 있는가?

---

## 4단계 — 재고 (모든 사이클을 연결하는 허브)

**개념:** 재고는 **물리적(physical)** 값과 **재무적(financial)** 값이 따로 움직인다. 이게 D365 재고관리 전체를 관통하는 핵심 개념.

| 확인 대상 | 경로 |
|---|---|
| 재고 트랜잭션 조회 | Inventory management > Inquiries > Transactions |
| 재고 현재 값 | Item 페이지 > On-hand inventory |

**체크:** 입고 직후와 인보이스 처리 후, 재고의 물리적 값과 재무적 값이 각각 어떻게 달라지는지 실제로 조회해서 눈으로 확인했는가?

---

## 5단계 — 생산 (제조업 고객 대응 시 필수, 지금은 개념만)

**흐름:**
```
BOM(자재명세서) + Route(공정경로) → 생산주문 생성 → 자재 소비 → 완성품 보고
```

| 개념 | 의미 |
|---|---|
| BOM | 완제품 하나를 만드는 데 필요한 원자재 구성 |
| Route | 원자재가 완제품이 되기까지 거치는 공정 순서 |
| WIP | 원자재도 완제품도 아닌, 생산 중인 상태의 재고 |

**체크:** 지금은 실습 없이 개념만 이해. 원자재 → WIP → 완제품 흐름을 한 줄로 설명 가능한지만 확인.

---

## 6단계 — 회계 (모든 것의 종착지)

**개념:** 입고, 출고, 인보이스, 지급 등 거의 모든 트랜잭션은 결국 Voucher(전표)로 기록되고 General Ledger에 전기된다.

| 확인 대상 | 경로 |
|---|---|
| 특정 트랜잭션의 회계 전기 확인 | 각 문서(인보이스 등) 화면 > Voucher 조회 |

**체크:** 오늘 만든 PO/SO 사이클 각 단계마다, Voucher transaction을 열어서 실제로 어떤 계정에 얼마가 찍히는지 최소 한 번씩 확인했는가?

---

## 7단계 — 통제 장치 (Workflow / Security) — 전 모듈 공통

**개념:** 특정 모듈 소속이 아니라, 모든 모듈에 공통으로 적용되는 "통제 레이어".

| 항목 | 경로 |
|---|---|
| Change management 활성화 | Procurement and sourcing > Setup > Procurement and sourcing parameters |
| 워크플로우 설정/승인자 | Setup > Procurement and sourcing workflows |

**체크:** Change management를 켜기 위한 두 가지 필수 조건(파라미터 활성화 + 워크플로우 Active)을 설명할 수 있는가? (이미 실습 완료)

---

## 최종 사고 회로 — 컨설턴트로서 갖춰야 할 습관

고객이 업무를 설명하면, 아래 4단계 질문이 자동으로 떠올라야 한다:

```
1. 이게 7가지 흐름(마스터데이터/구매/판매/재고/생산/회계/통제) 중 어디에 해당하나?
2. D365 표준 기능으로 되나, 커스터마이징이 필요한가?
3. 재고/회계에 어떤 영향을 주는 지점인가?
4. 물리적 확정과 재무적 확정, 어느 단계에서 갈리는가?
```

---

## 현재 진행 상황 체크리스트

- [x] 2단계 구매 사이클 — 개념 학습 + Change management 실습 완료
- [ ] 1단계 마스터 데이터 — Customer/Vendor/Item 등록 (진행 중)
- [ ] 3단계 판매 사이클
- [ ] 4단계 재고 — Inventory transactions 직접 조회
- [ ] 5단계 생산 — 개념만
- [ ] 6단계 회계 — Voucher transaction 확인 습관
- [ ] 7단계 통제 — 이미 실습, 복습만

---

*작성일: 2026-07-15 / 참고: 선임 컨설턴트 지시사항 + 2.1~2.4 학습 내용 종합*
