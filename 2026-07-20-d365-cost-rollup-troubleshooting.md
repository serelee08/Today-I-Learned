# 대표님 미팅 정리 (오후) — Product/Costing/Planning 개념

**날짜:** 2026-07-20 (오후)
**참석:** 대표님, 이지애

---

## 1. 용어 정리

**TDD (Technical Design Document)**
- 어제 나온 FDD(Functional Design Document, 기능 설계서)의 짝 문서
- FDD가 "무엇을 만들지"(요구사항→기능 설계)라면, TDD는 "어떻게 만들지"(기술 구현 방법, 테이블/필드/커스터마이징 설계)를 정리한 문서

---

## 2. Variant Configuration (변형 상품 구성)

Chapter 2에서 다룬 내용과 동일한 개념:
```
Product master  = 변형 상품의 "틀" (예: TV)
Product variant = 그 틀에서 나온 실제 변형 (예: 검정 46인치)
Configuration technology = 변형을 어떻게 만들지 정하는 방식
  - Predefined variant: 미리 정의된 차원(색상/사이즈) 조합
  - Dimension-based: 판매 시점에 차원 값 선택
  - Constraint-based: Product Configurator로 복잡한 조합 구성
```

**Product variant를 "코드로만 찾는지"에 대한 확인**
→ 정확히는 **"코드로만"은 아님.** Variant 고유번호(코드)로도 검색 가능하지만, 화면상에서는 Master + 차원값(색상=Black, 사이즈=46 등) 조합으로도 필터링/검색 가능함. 코드 검색은 여러 방법 중 하나.

**Product variant vs Product master 차이**

| 구분 | Product master | Product variant |
|---|---|---|
| 성격 | 틀/템플릿 | 실제 개별 상품 |
| 법인에 직접 릴리즈 | 가능 | 불가 (master를 통해서만 릴리즈됨) |
| 차원 값 | 미확정 (여러 조합 가능) | 확정된 값 (예: 검정, 46인치) |
| 거래(PO/SO) 사용 | 불가 (틀 자체는 거래 대상 아님) | 가능 (실제 거래 단위) |

---

## 3. Released Product 화면 구성 (가볍게)

Released product 상세 화면은 여러 Fast tab(접이식 섹션)으로 구성됨 — 오늘 TEST001 실습에서 이미 다룬 것들:
```
General      = 기본 식별정보, Dimension group들
Purchase     = 구매 관련 기본값
Sell         = 판매 관련 기본값
Manage costs = 원가(Item group, Item price 등)
Manage inventory = 재고 단위, On-hand 등
Engineer     = BOM, Production type 등
```
→ 한 화면 안에 "이 상품에 대한 모든 설정"이 카테고리별로 접혀 들어가 있는 구조.

---

## 4. 재고 계층(Hierarchy) — Warehouse vs Location

```
일반적: Site > Warehouse 단위까지만 관리
소형/고가 품목: Site > Warehouse > Location(구역/선반)까지 세밀하게 관리
```
→ 관리 정밀도와 업무 부담은 반비례. 필요 이상으로 세밀하게 관리하면 오히려 비효율적이라, 품목 특성에 따라 계층 깊이를 다르게 가져감.

---

## 5. Manage costs 안 Item group (가볍게)

- Item group은 **이 품목이 거래될 때 자동으로 어느 회계 계정(재고자산, 매출원가 등)에 전기될지**를 결정하는 연결고리
- 즉 "이 물건을 사고팔 때 회계상 어디에 기록할지"를 미리 정해두는 설정
- 오늘 TEST001에서 "Services"로 되어있던 그 필드가 바로 이것

---

## 6. Categories, Attribute (요약)

```
Categories = 구매/보고 목적의 상품 분류 체계
            (예: RFQ 발송 시 카테고리별로 벤더 그룹핑, 지출 분석 등)

Attribute  = 상품에 붙는 커스텀 특성/검색용 필드
            (Product dimension인 색상/사이즈와는 다른 개념 —
             검색/필터링을 돕는 부가 정보에 가까움)
```

---

## 7. Financial Inventory vs Physical Inventory

```
Physical inventory(물리적 재고) = 실제로 창고에 들어오고 나간 수량
                                (Product receipt/Packing slip 시점에 반영)

Financial inventory(재무적 재고) = 회계적으로 확정된 가치
                                (Invoice 시점에 최종 반영, 
                                 그 전까지는 잠정(accrual) 상태)
```
→ 오늘 Product Receipt에서 배운 "잠정 vs 확정" 개념과 동일한 원리.

---

## 8. MRP vs MPS

```
MRP (Material Requirements Planning, 자재소요계획)
= 수요 대비 "무엇을, 언제, 얼마나 구매해야 하는지" 계산
= 대표님 설명: 구매 모듈 쪽에 가까움

MPS (Master Production Schedule, 기준생산계획)
= "무엇을, 언제 생산할지"의 상위 계획
= 대표님 설명: 생산 모듈 쪽에 가까움
```
→ 둘 다 D365에서는 "Master planning" 모듈 안에 통합되어 있지만, 실무적으로 MRP는 구매(자재 조달), MPS는 생산(제조 일정) 쪽에 방점이 있다는 설명.

**ERP의 원조는 MRP** — ERP 역사적으로 1960~70년대 MRP(자재소요계획) → 1980년대 MRP II(생산능력/일정까지 확장) → 1990년대 이후 ERP(재무/인사 등 전사로 확장)로 발전. 지금 배우는 ERP의 뿌리가 MRP라는 설명.

---

## 9. BOM (Bill of Materials) — 가볍게

```
BOM = 완제품 1개를 만드는 데 필요한 
      부품/재료의 목록과 수량을 정의한 "레시피"
```
- 원가 계산(부품 원가 합산)과 생산(자재 소모 계획) 양쪽에 쓰임
- 어제/오늘 실습한 TEST001에서 "Production type: BOM"으로 잘못 설정돼서 문제 생겼던 그 개념

---

## 10. Inventory Dimension Group (이해 정리)

```
Inventory dimension group = 앞서 배운 3가지 그룹의 "상위 개념/총칭"

  ├─ Product dimension group (Size, Color, Configuration)
  ├─ Storage dimension group (Site, Warehouse, Location, Pallet ID)
  └─ Tracking dimension group (Serial number, Batch number)
```
→ 이 3개를 합쳐서 부르는 말이 Inventory dimension. 오늘 TEST001에서 각각 따로 설정했던 것들이 사실 이 큰 틀 안에 속함.

---

## 11. Default Order Setting — 가격 관련 확인

**정확히 하면:** Default order settings는 **가격 자체를 설정하는 곳이 아니라**, 주문 시 기본으로 들어갈 값들(기본 Site/Warehouse, 최소/최대 주문수량, 배수단위 등)을 미리 정해두는 곳.

**가격은 별도로 두 군데서 관리됨:**
```
Item price (Manage costs)  → 원가(Cost) 설정
Trade agreement            → 벤더별/고객별 실제 거래가격(할인 포함) 설정
```
→ Default order settings와 가격 설정은 서로 다른 화면이라는 점, 확인 필요.

---

## 12. Purchase Lead Time (쉬운 설명)

```
Purchase lead time = "주문하고 나서 실제로 물건이 도착하기까지 걸리는 시간"
```
- 예: Lead time 7일 → 오늘 주문하면 시스템이 자동으로 "7일 후 도착 예정"으로 날짜 계산
- Master Planning(MRP)에서 "언제 미리 주문 넣어야 하는지" 역산할 때 핵심 변수로 쓰임

---

## 13. Costing Version이란?

```
Costing version = 특정 시점 기준의 원가 "묶음"을 관리하는 버전
```
- 예: "2026년 7월 표준원가 버전", "2026년 8월 예상원가 버전"처럼 여러 버전을 만들어두고 비교/시뮬레이션 가능
- 아까 Item price 등록할 때 봤던 "Current fiscal period / Next fiscal period" 선택이 바로 이 Costing version 개념

**Item price가 Agreement에서 쓰이는 건가? — 더블체크 결과**

→ **아니야, 이 부분은 정정이 필요해.** Item price(Cost)와 Trade agreement(가격 협상)는 서로 다른 기능이야:
```
Item price   = 회사 내부 "원가" 관리용 (재고평가, 회계용) — Cost Management 영역
Agreement    = 벤더/고객과의 "거래가격" 협상용 (할인, 계약단가) — Trade agreement 영역
```
Item price가 Agreement 안에서 "쓰인다"기보다는, **서로 독립적인 두 개의 다른 가격 체계**로 이해하는 게 정확함.

---

## 14. Unit of Measure (UOM) 간단정리

```
UOM = 이 상품을 세는 "단위" (ea, box, kg, liter 등)
```
- 구매는 Box 단위, 재고는 EA(개) 단위, 판매는 다시 Box 단위처럼 상황별로 다른 단위 쓸 수 있음
- 이때 단위 간 환산이 필요 → 아래 13번(Unit conversion)과 연결

---

## 15. Organization Administration > Setup 의미

```
Organization administration = 조직 구조 자체를 관리하는 모듈
Setup 안에는:
  - Legal entities 등록
  - Operating units(Cost center, Business unit 등) 등록
  - Number sequences(채번 규칙) 설정
  - Calendar(회계기간 캘린더) 설정
  - Organizational hierarchies 구성
```
→ "회사 조직도 자체를 시스템에 세팅하는" 관리 영역.

---

## 16. Unit Conversion 간단요약

```
Unit conversion = 서로 다른 단위 간의 환산 비율을 정의
예: 1 Box = 12 EA
```
- 구매는 Box로 주문했는데 재고는 EA로 관리하면, 시스템이 이 환산 비율로 자동 계산
- 오늘 UOM 개념(14번)과 세트로 이해하면 됨

---

## 오늘 오후 숙제 (Action Items)

| # | 과제 |
|---|---|
| 1 | Batch number 정의 파악 |
| 2 | Lot number(롯드번호) 정의 파악 |
| 3 | Agent 관련 유튜브 찾아보기 |
| 4 | 재고평가방법 정리 (FIFO/LIFO/표준원가/이동평균 등 방법별로) |
| 5 | Chapter 2 Lab 직접 해보기 (Product master/variant 생성 및 릴리즈 실습) |

---

**Author:** [@serelee08](https://github.com/serelee08)
