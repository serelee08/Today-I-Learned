# 대표님 미팅 정리 — Chapter 3 Inventory Management Setup

**날짜:** 2026-07-21
**참석:** 대표님, 이지애

---

## 1. Chapter 3 (Inventory Management Setup) 프리뷰

대표님이 Chapter 3 전반을 미리 설명해주심.

---

## 2. Item Group 생성 — AR/AP 부분이 여기서 전부 생성된다

**대표님 설명:** Item group을 만들 때 AR(Accounts Receivable, 매출채권)과 AP(Accounts Payable, 매입채무) 관련 계정이 전부 여기서 설정된다.

**의미:**
- Item group은 단순한 "상품 분류"가 아니라, **"이 상품을 사고팔 때 회계 장부의 어느 계정에 기록할지"를 결정하는 연결고리**
- Item group의 Posting 설정 안에 탭이 나뉘어 있음:
  - **Sales order 탭** = AR(매출채권) 관련 계정 설정 (Revenue, Packing slip, Discount 등)
  - **Purchase order 탭** = AP(매입채무) 관련 계정 설정 (Purchase expenditure, Product receipt 등)
  - **Inventory 탭** = 재고 자체의 회계 계정 설정
- 즉 **상품이 거래될 때 자동으로 어떤 GL(총계정원장) 계정에 전기될지를, Item group 하나에서 전부 매핑하는 구조**

---

## 3. Sales Order Fast Tab — Account Type 정리

**경로:** Item group > Posting > Sales order 탭

| Account Type | 언제 발생하는지 |
|---|---|
| **Packing slip** | 출고(Packing slip) 처리 시 → 물리적 재고 감소 기록 |
| **Packing slip offset** | Packing slip의 상대 계정 (차변/대변 맞추기용) |
| **Issue** | 재고가 출고될 때 발생하는 재고자산 감소 |
| **Consumption** | 생산 공정에서 자재를 소비할 때 |
| **Revenue** | 매출 인보이스 발행 시 → 매출(수익) 인식 |
| **Discount** | 고객에게 할인 적용 시 → 할인 비용 계정 |
| **Commission** | 영업 수수료 발생 시 |
| **Commission offset** | 수수료의 상대 계정 |
| **Deferred revenue on delivery** | 출고는 했지만 아직 인보이스 미발행 → 이연수익으로 잠정 처리 |
| **Deferred revenue offset on delivery** | 이연수익의 상대 계정 |
| **Deferred sales tax on delivery** | 출고 시점 잠정 세금 계정 |

---

## 4. Invoice Journal — Inventory Profit / Inventory Loss 용어

| 용어 | 의미 |
|---|---|
| **Inventory profit (재고 이익)** | 재고 마감(Inventory close) 시, 입고 원가보다 출고 원가가 낮게 정산돼서 발생하는 이익. 예: 100원에 산 물건의 원가가 재계산으로 90원으로 조정 → 10원 이익 |
| **Inventory loss (재고 손실)** | 반대로 출고 원가가 높게 정산돼서 발생하는 손실. 예: 100원에 산 물건의 원가가 110원으로 조정 → 10원 손실 |

**물류에서의 의미:** 실제 물건이 없어지거나 생기는 게 아니라, **원가 재계산(재평가) 과정에서 발생하는 회계적 차이**. 재고 마감(Inventory close) 시점에 FIFO/LIFO 등 평가 방법에 따라 원가가 재배분되면서 자동으로 발생함.

---

## 5. Shipment은 Sales Order에서 사용

**대표님 설명:** Shipment(출하)은 Sales Order 프로세스에서 사용된다.

**부가 설명:**
- 구매(Purchase) 프로세스에서는 "입고(Receipt)" → Product Receipt
- 판매(Sales) 프로세스에서는 "출고(Shipment)" → Packing Slip
- 이 둘은 정확히 대칭 관계:
  - Purchase: PO → Product Receipt(입고) → Vendor Invoice
  - Sales: SO → Packing Slip(출고/Shipment) → Customer Invoice

---

## 6. Released Product 필수 5가지 설정

**대표님 설명:** General 탭에서 가장 중요한 5가지를 기본값으로 반드시 설정하라고 하심.

| 순서 | 필수 설정 | 역할 |
|---|---|---|
| 1 | **Item model group** | 재고 평가 방법(FIFO/Standard cost 등), 물리적/재무적 전기 여부 결정 |
| 2 | **Item group** | 거래 시 자동 전기될 GL 계정 매핑 (AR/AP 계정 포함) |
| 3 | **Storage dimension group** | Site/Warehouse/Location 추적 여부 결정 |
| 4 | **Tracking dimension group** | Serial/Batch number 추적 여부 결정 |
| 5 | **Product dimension group** | Color/Size/Configuration/Style/Version 변형 여부 결정 (Product master에만 해당) |

---

## 7. Stocked vs Non-stocked 옵션

| 구분 | Stocked (재고추적 상품) | Non-stocked (재고미추적 상품) |
|---|---|---|
| 실물 재고 추적 | O (On-hand에 수량 잡힘) | X (On-hand에 안 잡힘) |
| 예약(Reservation) | 가능 | 불가 |
| Serial/Batch 추적 | 가능 | 불가 |
| 재무상 분류 | 현재자산(재고자산) | 바로 비용(손익) 처리 |
| 사용 예시 | 원자재, 완제품, 반제품 | 서비스, 소모품, 컨설팅비 |

---

## 8. Item Model Group — Post Physical/Financial Inventory

| 설정 | 의미 | 실제 예시 |
|---|---|---|
| **Post physical inventory** | 물리적 거래(입고/출고) 시 GL에 전기할지 여부 | 체크하면: Product Receipt 처리할 때 "재고자산(잠정)" 계정에 자동 전기됨. 체크 안 하면: 재고 수량만 변하고 회계엔 아무 기록 안 남음 |
| **Post financial inventory** | 재무적 거래(인보이스) 시 GL에 전기할지 여부 | 체크하면: Vendor Invoice 처리할 때 "매입채무(AP)" 계정에 자동 전기됨. 보통 이건 거의 항상 체크됨 |

---

## 9. FEFO (First Expired, First Out) 및 Physical Negative Inventory

**FEFO = First Expired, First Out (유통기한 빠른 것부터 먼저 출고)**
- FIFO(먼저 들어온 것부터)와 다름 — FEFO는 "들어온 순서"가 아니라 **"유통기한이 가장 빨리 도래하는 것"**부터 출고
- 식품, 의약품, 화학품 등 유통기한이 중요한 상품에 사용
- D365에서는 Item model group의 **"FEFO date-controlled"** 옵션을 켜고, **"Pick criteria"** 필드에서 설정
- **FEFO는 원가 평가 방법(Costing)이 아니라 "예약/피킹 전략(Reservation technique)"**임에 주의

**Physical Negative Inventory = 물리적 마이너스 재고 허용 여부**
- 체크하면: 실제 재고가 0인데도 출고(판매/소비) 처리를 허용함. 예: 재고 0인데 판매주문 출고 가능
- 체크 해제하면: 재고 없으면 출고 자체를 시스템이 차단
- FEFO와의 관계: Physical negative inventory가 켜져 있으면, FEFO 순서를 무시하고 재고가 없는 상태에서도 출고될 수 있어서 FEFO의 의미가 약해질 수 있음. **유통기한 관리가 중요한 상품이면 Physical negative inventory를 꺼두는 게 안전**

---

## 10. 제품 유형 용어 정리

| 용어 | 의미 |
|---|---|
| **제품 (Product)** | 판매 가능한 완성된 상품 (= 완제품과 같은 의미로도 쓰임) |
| **상품 (Goods/Merchandise)** | 유통/판매 목적의 물품 (제품보다 넓은 의미) |
| **완제품 (Finished goods)** | 생산 공정을 모두 거쳐 판매 가능한 상태의 제품 |
| **반제품 (Semi-finished goods)** | 생산 도중인 미완성 제품, 추가 가공이 필요 (= WIP, Work in Process) |
| **원재료 (Raw materials)** | 생산의 주요 투입 자재 (예: 철강, 목재, 원유) |
| **부재료 (Sub-materials/Auxiliary materials)** | 생산 보조 자재, 제품에 직접 포함되지 않거나 소량 사용 (예: 접착제, 나사, 포장재) |

---

## 11. Production Type 관련 용어

| 용어 | D365에서의 의미 |
|---|---|
| **BOM** | 여러 부품을 조립해서 만드는 완제품 (Bill of Materials) |
| **Formula** | 화학/식품 등 배합 비율로 만드는 공정 제품 (Process manufacturing) |
| **Planning item** | 실제 생산/구매 대상이 아니라 MRP 계획 목적으로만 존재하는 가상 품목 |
| **Co-product** | 하나의 생산 공정에서 의도적으로 같이 나오는 복수 산출물 |
| **By-product** | 주 생산물의 부산물 (의도하지 않았지만 함께 나오는 것) |
| **None** | 생산 방식 없음, 단순 매입/판매 상품 |

---

## 12. Taxation — 기본값 ALL 지정

**대표님 설명:** Taxation 관련 설정은 항상 기본값으로 "ALL" 지정하라고 하심.
- Item sales tax group 등에서 "ALL"로 설정하면 모든 세금 규칙이 적용되는 기본 상태
- 특정 면세/특수세율 상품이 아닌 이상, 실습 단계에서는 ALL로 두는 게 안전

---

## 13. Unit Conversion (단위 환산)

- 구매/재고/판매 단계마다 서로 다른 단위를 쓸 수 있음
- 예: 구매는 Box 단위, 재고는 EA(개) 단위, 판매는 Pack 단위
- Unit conversion = 이 단위 간 **환산 비율을 정의**하는 설정 (예: 1 Box = 12 EA)
- 경로: Product information management > Products > Released products > Unit conversions

---

## 14. Overdelivery / Underdelivery

| 용어 | 의미 |
|---|---|
| **Overdelivery (초과 납품)** | 주문 수량보다 더 많이 입고/출고되는 것. **Percentage(%)로 허용 범위를 설정**. 예: 10% overdelivery 허용 → 100개 주문에 110개까지 입고 가능 |
| **Underdelivery (부족 납품)** | 주문 수량보다 적게 입고/출고되는 것. 마찬가지로 **%로 허용 범위 설정**. 예: 5% underdelivery 허용 → 100개 주문에 95개만 와도 시스템이 "완료"로 처리 가능 |

---

## 15. Pick Criteria / Validate Batch Expiration

| 용어 | 의미 |
|---|---|
| **Pick criteria** | 출고(피킹) 시 어떤 기준으로 재고를 선택할지 정하는 규칙. FEFO 사용 시 여기서 "유통기한 기준" 선택 |
| **Validate batch expiration** | Batch(배치) 상품의 유통기한이 지났는지 시스템이 자동으로 검증하는 기능. 체크하면: 유통기한 지난 배치는 출고/소비 시 시스템이 차단 또는 경고 |

**둘 다 Item model group > Physical update 관련 설정에 위치**

---

## 오늘 받은 과제 (Action Items)

| # | 과제 | 상태 |
|---|---|---|
| 1 | Product Information Management Lab 2-1, 5개 만들기 | ⬜ 예정 |
| 2 | 회계 책 구독 계획 세우기 | ⬜ 예정 |
| 3 | Service item 만들어보기 | ⬜ 예정 |
| 4 | Chapter 3-1 Lab 실습 | ⬜ 예정 |
| 5 | Site와 Warehouse 생성 | ⬜ 예정 |

---

**Author:** [@serelee08](https://github.com/serelee08)
