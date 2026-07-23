# 대표님 미팅 정리 및 업무 기록 — 2026-07-23

**작성자:** 이지애

---

## 1. MES (Manufacturing Execution System) — 개념
MES = 공장 현장(Shop floor)에서 실제 생산이 진행되는 과정을 
     실시간으로 관리·모니터링하는 시스템

ERP(계획 단계) ↔ MES(실행 단계) ↔ 실제 설비/현장
```

**핵심 역할:**
- 생산 작업 지시(Work order) 진행상황 실시간 추적
- 설비 가동 상태, 생산 수량, 품질 검사 결과 기록
- ERP가 "계획"한 것을, MES가 "현장에서 실제로 실행"되게 연결하는 다리 역할

**D365 연결점:** AX2012 매핑 자료의 "ME Ch.3: Production Control with Manufacturing Execution" 챕터가 이 개념을 다룸 (학습 플랜 8.3 항목)

---

## 2. Location 생성 에러 해결

**문제 상황:**
```
Inventory locations 화면에서 Location 신규 등록 시도
→ Location 필드가 입력 자체가 안 되는(비활성화) 상태
→ Save 시도할 때마다 
  "Field 'Location' must be filled in" 에러 반복
```

**해결 과정:**
```
1. General 섹션 안의 "Manual update" 토글을 켬 (No → Yes)
   → 이 토글을 켜자 Location 필드가 활성화(입력 가능)됨

2. "Other" 섹션의 "Destination location" 필드도 
   필수(빨간 테두리)로 표시되어 있어서 함께 채움

3. Location 코드 입력 → Save → 정상 저장 확인
```

**결과:** Location 정상 등록 완료, 어제(7/22) 막혔던 CHAPTER2-1LAB3 Product Receipt 재시도 가능한 상태로 해결

---

## 3. Excel Add-in 사용법

**핵심 개념 — "Open in Excel" vs "Export to Excel"**

```
Export to Excel
= 데이터를 엑셀로 "복사"만 하는 정적(static) 내보내기
= D365랑 연결이 끊긴 일회성 스냅샷

Open in Excel
= D365 데이터와 "실시간으로 연결"되는 동적 내보내기
= 엑셀에서 수정한 내용을 다시 D365로 반영(Publish) 가능
```

**사용 방법:**
```
1. D365 화면(리스트 페이지)에서 "Open in Microsoft Office" 버튼 클릭
2. "Open in Excel" 섹션에서 원하는 항목 선택 → Download
3. 다운로드된 엑셀 파일 열기 → "Enable Editing" 클릭
4. 우측에 Dynamics 연결 패널(Connector)이 나타나며 로그인 요청
5. 로그인하면 실제 D365 데이터가 엑셀에 로드됨
6. 엑셀에서 데이터 수정 (필터링, 피벗, 계산식 추가 등 자유롭게 가능)
7. 우측 패널에서 "Update"(또는 Publish) 클릭
   → 수정한 내용이 D365로 다시 반영됨
```

**주의점:**
- 사용자 보안 권한 내에서만 데이터 조회/수정 가능
- 대량 수정(Mass update)/삭제 시 주의 필요 (검증은 거치지만 되돌리기 어려움)

---

## 4. 과제 — Add lines "Copy" 기능

**핵심 개념: Add line vs Add lines**
```
Add line  = 라인 하나씩 수동으로 직접 추가
Add lines = 여러 상품을 목록에서 필터링해서 
           한 번에 여러 줄 추가 (직접 만드는 상품 목록 기반)
```

**"Copy" 기능 — 여러 아이템을 한 번에 가져오는 방법**
```
경로: PO 화면 > 리본 "Purchase order" 탭 > "Copy" > "From all"
```

**동작 방식:**
```
1. "From all" 클릭하면, 복사해올 수 있는 원본 문서 목록이 뜸
   (다른 기존 PO, 또는 Sales order 등)
2. 원본 문서에서 원하는 라인들을 체크박스로 선택
3. Parameters 섹션에서:
   - Quantity factor: 수량을 몇 배로 가져올지 
     (예: 2 입력하면 원본의 2배 수량으로 복사됨)
   - Invert sign: 수량의 부호를 반전 
     (반품/크레딧 처리 시 사용)
4. 실행하면 선택한 라인들이 전부 
   한 번에 현재 PO에 복사되어 들어감
```

**요약:** "Copy" 기능은 매번 Item을 하나하나 손으로 검색해서 추가하는 대신, **이미 존재하는 다른 주문서(PO/SO)의 라인들을 통째로 복사해와서** 여러 아이템을 한 번에 채워 넣을 수 있게 해주는 기능. 반복적인 주문(같은 벤더한테 매번 비슷한 품목을 주문하는 경우)에서 특히 유용함.

---

## 오늘 진행상황 (Action Items)

| # | 항목 | 상태 |
|---|---|---|
| 1 | Location 마스터 데이터 등록 (Manual update 이슈 해결) | ✅ 완료 |
| 2 | Chapter 3-1 Lab (Item model group "DVD" 생성·적용) | ✅ 완료 |
| 3 | Service item 생성 (Warranty, Non-stocked 확인) | ✅ 완료 |
| 4 | Item group Account type 구조 확인 (Monitor 그룹) | ✅ 완료 |
| 5 | Excel Add-in 사용법 조사 | ✅ 완료 |
| 6 | Add lines "Copy" 기능 조사 | ✅ 완료 |
| 7 | PIM Lab 2-1 나머지 2개 (4/5, 5/5) | ⬜ 예정 |
| 8 | 재고평가방법 정리 (FIFO/LIFO/표준원가/이동평균) | ⬜ 예정 |
| 9 | Number sequence setup 직접 실습 | ⬜ 예정 |
| 10 | 전체 Modules 기능 파악 | ⬜ 예정 |
| 11 | 회계기초 탈출기 Day3 학습 | ⬜ 예정 |
| 12 | Chapter 5 스터디 | ⬜ 예정 |

---

**Author:** [@serelee08](https://github.com/serelee08)
