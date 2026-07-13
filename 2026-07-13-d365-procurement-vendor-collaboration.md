# 2026-07-13 — D365 F&SCM: Procure-to-Purchase 프로세스 & Vendor Collaboration

## 오늘 한 일

입사첫날 PROC(Procurement) Ch.1/6/7과 VMC(Vendor Collaboration) Module 1~5를 학습했다.

1. **2.1 Configure and perform the procure-to-purchase process**
   구매요청(PR) → 구매주문(PO) → 입고(Product Receipt) → 송장(Vendor Invoice) 전체 흐름 학습
2. **2.2 Configure and manage vendor collaboration**
   USMF 데모 환경에서 Vendor Collaboration 유저 프로비저닝 실습 (Microsoft Learn 공식 hands-on lab)

---

## 오늘 배운 것 (기술적으로)

### 1. PR과 PO는 다른 문서다

- **PR(Purchase Requisition)**: 내부 승인 요청. 벤더에게 나가는 문서가 아니라 "사도 되냐"는 사내 결재 문서.
  - Purpose는 두 종류로 나뉜다: **Consumption**(내부 사용, 항상 PO로 충족) vs **Replenishment**(재고 보충, PO/이전주문/생산주문/칸반 중 무엇으로든 충족 가능, 금액이 아니라 수량으로 표현됨)
  - 상태: `Draft → (workflow review) → Approved`
- **PO(Purchase Order)**: 승인된 PR을 바탕으로 벤더에게 실제로 나가는 공식 주문서.
  - 상태: `Approved → Confirmed → Received → Invoiced`

### 2. 입고(Product Receipt)와 송장(Vendor Invoice)의 차이 — 오늘의 핵심

이 둘을 혼동하면 안 된다.

| | Product Receipt | Vendor Invoice |
|---|---|---|
| 재고 수량(physical) | 갱신됨 | 변화 없음 |
| 재고 금액(financial) | **변화 없음** | 갱신됨 |
| 벤더 미지급 잔액 | 변화 없음 | 증가 |
| GL(총계정원장) 전기 | 없음 | 전기됨 |

물건이 창고에 들어와도(Product Receipt) 회계장부는 아직 안 움직인다. **Vendor Invoice를 Post 해야** 비로소 재무 값이 갱신되고, 벤더에게 줄 돈이 잡히고, GL에 반영된다. `3-way matching`(PO ↔ 입고 ↔ 인보이스 대조)이 여기서 결제 전 최종 검증 역할을 한다.

### 3. Vendor Collaboration = D365가 제공하는 벤더용 셀프서비스 포털

EDI(시스템 간 자동 데이터 교환)가 구축되지 않은 벤더를 위한 대안. 벤더가 직접 D365 안의 포털에 로그인해서:
- PO를 **Accept / Reject / Accept with changes** 중 선택해 응답
- RFQ(견적요청)에 입찰
- 인보이스를 직접 제출

EDI는 D365의 기본 내장 기능이 아니라, 회사가 별도로 구매해서 붙이는 제3자(ISV) 솔루션이라는 것도 오늘 알았다 (예: TrueCommerce, SPS Commerce, SEEBURGER). ANH코리아 sandbox에는 SEEBURGER가 붙어 있었다.

### 4. Vendor Collaboration 유저 프로비저닝 실습 (USMF)

공식 절차:
```
1. Procurement and sourcing > Vendors > All vendors
2. Vendor(예: 1001) 선택 → General 탭에서
   Collaboration activation = "Active (PO is auto-confirmed)" 설정
3. Action Pane > Set up > Contacts > Add contacts → 담당자 등록
4. Action Pane > Set up > Contacts > View contacts (별도 페이지로 이동)
5. 그 페이지에서 Provision vendor user 클릭
   - Email (개인 이메일 @gmail/@hotmail 불가, 회사 도메인 형식 필요)
   - Business justification 입력
   - Vendor collaboration access allowed 체크
   - Assign user roles에서 Vendor(external)/Vendor admin(external) 역할 체크
6. Submit → 워크플로우로 승인 대기 상태 진입
```

핵심은 **"Provision vendor user" 버튼이 Vendor 메인 페이지가 아니라, Contacts → View contacts로 들어간 별도의 연락처 목록 페이지 안에 있다는 것**이다. Vendor 메인 페이지 Action Pane만 계속 뒤지면 못 찾는다.

---

## 막혔던 점 & 해결

| 막힌 점 | 원인 | 해결 |
|---|---|---|
| Vendor 1001 검색해도 안 뜸 | 회사가 ANH코리아 sandbox(자체 회사코드)로 설정되어 있었고, 이 실습은 Microsoft 표준 USMF 데모 데이터 전제 | 상단 회사 선택기에서 USMF로 전환 |
| "Provision vendor user" 버튼을 못 찾음 | Vendor 메인 페이지 Action Pane에서만 계속 찾음 | 공식 문서 확인 결과, Contacts 드롭다운의 **"View contacts"**로 들어가야 하는 별도 페이지였음 |
| Role 부여 방식이 헷갈림 | Privilege를 직접 Role에 넣는 작업(Security configuration)과, 이미 만들어진 Role을 유저에게 배정하는 작업(Provision vendor user)을 혼동함 | 전자는 관리자가 사전에 세팅해두는 것, 후자는 오늘 실습 범위. 둘은 다른 화면, 다른 목적 |

---

## 더 공부할 것 (솔직한 메모)

- [ ] Purchasing policy에서 "PO creation and demand consolidation rule" 설정법 — PR→PO 자동 생성 조건
- [ ] RFQ(견적요청) 프로세스 전체 흐름 — PR에서 RFQ로 넘어가는 지점
- [ ] Extensible Data Security(XDS) 정책 구조 — Vendor Collaboration이 벤더별 데이터를 어떻게 격리하는지
- [ ] 부분 입고(partial receipt), 부분 송장(partial invoice) 처리 시나리오
- [ ] Security configuration에서 Role ↔ Duty ↔ Privilege 계층을 실제로 새로 만들어보기 (오늘은 기존 Role만 배정, Privilege 직접 구성은 아직 안 해봄)

---

**Author:** [@serelee08](https://github.com/serelee08)
