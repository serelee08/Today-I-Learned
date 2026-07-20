# 대표님 미팅 정리 — D365 용어 및 모듈 구조

**날짜:** 2026-07-20
**참석:** 대표님, 이지애

---

## 1. 오늘 학습(AX2012 WN_SCM Ch.1 Overview)에서 나온 용어 설명

대표님이 챕터에 나온 용어들을 실무 관점에서 풀어서 설명해주심.

| 용어 | 대표님 설명 |
|---|---|
| TCO (Total Cost of Ownership) | 도입부터 유지보수, 업그레이드까지 전체 생애주기 비용 |
| SRM (Supplier Relationship Management) | 공급업체 관계 관리 |
| HRM (Human Resources Management) | 인사 관리 |
| CRM (Customer Relationship Management) | 고객 관계 관리 |
| Dimension entry display | **"관리항목"이라고 이해하면 됨** (재무/원가 차원을 입력하는 화면) |
| FDD (Function Design Document) | 기능 설계 문서 — 프로젝트에서 요구사항을 기술 설계로 옮길 때 쓰는 산출물 |
| Financial consolidation process | 여러 법인/조직의 재무 데이터를 하나로 통합하는 과정 |

## 2. Organization Hierarchies

- Overview 챕터에 나온 "Organizational Hierarchies" 개념을 실제 시스템에서 View로 확인 가능하다는 것 설명받음
- Legal entity는 4글자 약어로 설정한다는 것 확인 (예: USMF)

## 3. Chapter 1 Excel Add-in 부분

- Overview Chapter 1의 Office Add-ins(Excel) 부분은 대표님이 직접 설명하기보다, **유튜브 검색해서 영상으로 확인**하는 걸 추천받음 → 별도로 찾아볼 것

## 4. 재고(Inventory) 관련

- **입고 전 대기 상태**: QA(품질검사) 등을 거치는 재고는 입고돼도 바로 사용 가능한 상태가 아니라 "대기" 상태로 남을 수 있음
- **Inventory management > On-hand**: 실물 재고를 의미. On-hand list에서 확인 가능

## 5. Chapter 2 (Product Information Management) — Supplier Risk Assessment

- Chapter 2 관련해서 **Supplier Risk Assessment** 안에 있는 **Performance(공급업체 성과 평가)** 기능들을 설명받음

## 6. 업무 영역별 모듈 매핑 (대표님 설명)

```
구매(Purchase) 영역   = Accounts Payable + Procurement and Sourcing
영업(Sales) 영역      = Accounts Receivable + Sales & Marketing
```

## 7. SCM에서 자주 쓰는 기능

- **Inventory management** (특히 Arrival overview — 입고 개요 화면)
- **Master planning** 기능들 — 각각 요약 및 개념 설명받음

---

## 오늘 받은 숙제 (Action Items)

| # | 과제 | 상태 |
|---|---|---|
| 1 | Receipt 후 Warehouse 등 수정하는 방법 찾기 | ✅ 완료 (Transfer journal로 해결, Serial number 이슈 확인 중) |
| 2 | Number sequence setup 시도해보기 | ⬜ 예정 |
| 3 | Agent(D365 AI 에이전트)에 대한 정보 찾아보기 | ✅ 완료 (2026 Wave 1 신규 기능, Account Reconciliation/Procurement Agent 등 확인) |
| 4 | 전체적인 Modules 기능 익히기 (용도 파악 수준) | ⬜ 진행 중 |
| 5 | Dashboard에 Workspace 기능 추가하는 방법 알아오기 | ⬜ 예정 |

---

**Author:** [@serelee08](https://github.com/serelee08)
