# 2026-06-20 — 컴포넌트 구조화 & Layout 패턴 & CatchCare 디자인

## 오늘 한 것
프로젝트 이름을 CatchCare로 정하고, 컴포넌트를 폴더로 구조화한 뒤 Layout 패턴과 민트 테마 디자인을 적용했다.

## 1. 폴더 구조화 (관심사 분리)
- components/layout/ : 공통 레이아웃 (Layout, Header, Footer)
- components/reservation/ : 예약 기능 (ReservationForm)
- 기능 성격에 따라 폴더 분리 → 유지보수 쉬움

## 2. Layout 패턴 (현업 표준)
- Layout이 Header + children + Footer를 묶음
- children = Layout 태그 사이 내용 → 헤더/푸터 사이에 끼워짐
- 헤더/푸터 고정, 가운데만 페이지마다 바뀜
- 공통 컴포넌트 중복 import 제거

## 3. import 경로 규칙
- ./ = 같은 폴더, ../ = 한 단계 위, ../../ = 두 단계 위
- 폴더 깊어지면 .. 개수 늘어남

## 4. CSS 분리 전략
- App.css = 공통 (색, 버튼, 칩, 카드)
- layout/Layout.css = 헤더/푸터
- reservation/ReservationForm.css = 예약 폼
- 각 컴포넌트가 자기 CSS import

## 5. 민트 테마 (CatchCare)
- 메인 #14B8A6, 딥틸 #0F766E, 배경 #F0FDFA
- 헤더: 민트 그라데이션 + 그림자
- 카드: 흰 배경 + hover 효과
- 상태 칩: 예약(민트)/완료(초록)/취소(빨강)

## 6. 오늘 겪은 버그 & 해결
① 파일명 공백: " Layout.jsx" 앞 공백 → ls -la로 발견 → mv로 수정
② import 경로: 폴더 이동 후 ../api → ../../api로 수정
→ "파일 없다" 에러는 이름/경로/공백 문제일 때 많음. ls -la로 확인하는 습관 중요.

## 회고
기능이 아니라 구조와 디자인을 다룬 날. 폴더 구조, Layout 패턴, CSS 분리는 실무 유지보수성을 좌우. 파일명 공백 버그를 ls -la로 직접 추적해 해결한 게 의미 있었다. 다음은 예약 폼 디자인 마무리.
