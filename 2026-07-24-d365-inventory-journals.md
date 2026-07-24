# 대표님 미팅 정리 — Chapter 8: Inventory Journals

**날짜:** 2026-07-24
**참석:** 대표님, 이지애

---

## 주제: Inventory Journals (재고 분개장) 5가지

**대표님이 재고 분개장 5가지를 요약해주심:**

```
1. Movement Journal (이동 분개장)
   = 재고를 다른 부서/용도로 이관, 
     상대계정(Offset account)을 직접 지정 가능
   = 예: 영업부서 샘플 반출, 부서간 이관

2. Inventory Adjustment Journal (조정 분개장)
   = 재고 이익/손실 조정
   = 상대계정 직접 지정 불가, Posting profile(자동 규칙)대로만 전기

3. Transfer Journal (이동/전송 분개장)
   = Warehouse/Batch/Variant 간 재고 이동
   = 한 줄 Post하면 입고+출고 트랜잭션 동시 발생

4. BOM Journal (자재명세서 분개장)
   = 조립형 완제품 생산 완료 보고
   = Post 시 완제품 입고 + 부품 출고 동시 발생

5. Counting Journal / Tag Counting Journal (실사 분개장)
   = 실제 재고조사 후 시스템 기록과 차이 조정
   = Tag counting은 수기 태그 방식, 
     Post하면 Counting journal이 자동 생성되는 2단계 구조
```

---

## 핵심 지시사항 — "항상 Transaction 확인하라"

```
분개장 Post한 뒤에는 반드시 
Inventory transactions(재고 거래 내역) 화면에서 
실제로 어떻게 반영됐는지 확인하는 습관을 가질 것

확인 경로:
Inventory management > Inquiries and reports > Transactions
또는 Journal Post 후 > Inventory > Transactions > 
Ledger > Financial voucher (차변/대변까지 확인)
```

**의미:** Post 버튼 누르고 끝내는 게 아니라, 실제로 재고 수량과 회계 전기(차변/대변)가 의도한 대로 정확히 반영됐는지 매번 검증하는 습관이 실무에서 중요하다는 강조.

---

## 오늘 실습과의 연결

```
오늘 Lab 8.1(Movement journal) 실습 중 
재고 있는 상품을 찾는 데 시간이 걸렸는데,
이 과정에서 On-hand inventory와 
Transactions 화면을 반복적으로 확인하며 
"실제 재고 vs 시스템 기록"을 대조하는 습관을 
직접 체득함 — 대표님 지시사항과 정확히 일치하는 경험
```

---

## 오늘 진행상황 (Action Items)

| # | 항목 | 상태 |
|---|---|---|
| 1 | Lab 5.1(Serial number 수동 등록, Approved vendor 에러 해결) | ✅ 완료 |
| 2 | Chapter 5, Chapter 8(Inventory Journals) 학습 | ✅ 완료 |
| 3 | Lab 8.1(Movement journal) 실습 | 🔄 진행 중 |
| 4 | CHAPTER2-1LAB3 Product Receipt 재시도 | ⬜ 예정 |
| 5 | 유지보수 사이트 파악(희성촉매, 인바디, 아이디룩) | ⬜ 예정 |
| 6 | 회계기초 탈출기 Day3 학습(재무제표의 작성) | ⬜ 예정 |
| 7 | PIM Lab 2-1 나머지 2개 완성(4/5, 5/5) | ⬜ 예정 |


---

**Author:** [@serelee08](https://github.com/serelee08)
