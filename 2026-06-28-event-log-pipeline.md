# 2026-06-28 — 이벤트 로그 파이프라인 (Docker + MySQL + 집계 + 시각화)

> 채용 과제: 웹 서비스 이벤트를 **생성 → 저장 → 집계 → 시각화**하는 데이터 파이프라인을
> 만들고 `docker compose up` 한 번으로 전체 스택이 돌아가게 구성.
> Stack: Python · MySQL 8 · Docker Compose · matplotlib

---

## 1. 오늘 배운 핵심 개념

### (1) 컨테이너 시작 순서 (startup ordering)
`docker compose up`을 하면 app 컨테이너와 db 컨테이너가 **거의 동시에** 뜬다.
문제는 MySQL이 초기화를 끝내기 전에 app이 먼저 떠서 DB 연결에 실패한다는 것.

해결:
- compose에서 db에 `healthcheck`를 걸고, app은 `depends_on: condition: service_healthy`로 **DB가 준비된 뒤에만** 시작.
- 그래도 안전하게, app 코드(`db.py`)에 **연결 재시도 루프**를 넣어 이중 방어.

> 면접 포인트: "왜 `depends_on`만으로 부족한가?" → 기본 `depends_on`은 *컨테이너가 떴는지*만 보지, *DB가 쿼리를 받을 준비가 됐는지*는 모른다. 그래서 `service_healthy` + 재시도가 필요.

### (2) JSON 통짜 저장 대신 필드 분리 + 인덱스
이벤트를 JSON 한 덩어리로 넣으면, 집계할 때마다 전부 읽어서 직접 파싱해야 한다.
컬럼으로 분리하면 `GROUP BY`로 **집계를 DB에 위임**할 수 있다.

집계 기준이 되는 컬럼에 인덱스를 걸었다:
- `event_type` (타입별 집계, 에러 비율)
- `user_id` (유저별 집계)
- `event_time` (시간대별 추이)

> 면접 포인트: "인덱스 단점은?" → 조회는 빨라지지만, INSERT/UPDATE마다 인덱스도 갱신해야 해서 **쓰기가 느려지고 저장공간을 더 쓴다.** 그래서 무조건 많이 거는 게 아니라 *집계/조회에 실제로 쓰는 컬럼에만* 건다.

### (3) 집계 쿼리 (GROUP BY)
4가지 분석을 SQL로 작성:
1. 이벤트 타입별 횟수 — `GROUP BY event_type`
2. 유저별 이벤트 수(상위 N) — `GROUP BY user_id ORDER BY cnt DESC LIMIT N`
3. 시간대별 추이 — `GROUP BY HOUR(event_time)`
4. 에러 비율 — `SUM(event_type='error') / COUNT(*)`

### (4) 배치 INSERT
이벤트 1000건을 한 건씩 INSERT하면 DB 왕복이 1000번.
`executemany`로 **묶어서(batch)** 넣으면 왕복 횟수가 크게 준다.
또 `%s` 파라미터 바인딩을 써서 SQL 인젝션을 막고, 드라이버가 prepared statement를 재사용하게 했다.

### (5) 헤드리스 환경의 matplotlib
컨테이너엔 화면(display)이 없어서, matplotlib이 GUI 창을 열려다 크래시 났다.
`matplotlib.use("Agg")`로 **GUI 없이 파일(PNG)로 바로 렌더링**하도록 해결.

---

## 2. 인프라 기초 (선택 과제)

### Kubernetes — Job vs Deployment
- DB는 계속 떠 있어야 하므로 **Deployment** + **Service**(고정 내부 주소).
- 이벤트 생성기는 한 번 돌고 끝나는 배치라 **Deployment가 아니라 Job**이 맞다.
  Deployment로 두면 끝없이 재시작된다.

### AWS — 서비스 역할 분리
- 정형 데이터(events 테이블) → **RDS for MySQL**
- 파일(차트 PNG 등 바이너리) → **S3** (DB에 BLOB로 넣으면 용량·백업·성능에 안 좋음)
- 컨테이너 실행 → **ECS Fargate** (서버 관리 없이 컨테이너만 실행)
- 로그/알람/관측 → **CloudWatch**

---

## 3. 막혔던 점 & 해결

| 막힌 점 | 원인 | 해결 |
|---------|------|------|
| app이 DB 연결 실패 | MySQL이 아직 준비 안 됨 | healthcheck + 재시도 루프 |
| 차트 생성 시 크래시 | 컨테이너에 디스플레이 없음 | `matplotlib.use("Agg")` |
| 집계가 느릴 수 있음 | 인덱스 없으면 풀스캔 | 집계 컬럼에 인덱스 |

---

## 4. 더 공부할 것 (솔직한 메모)

- [ ] 대용량일 때 `executemany` → `LOAD DATA`(bulk load)로 더 빠르게 하는 법
- [ ] 집계를 매번 계산하지 않고 **집계 테이블(롤업)**로 미리 쌓는 구조
- [ ] healthcheck의 `interval / retries`를 실제로 어떻게 튜닝하는지
- [ ] 이벤트가 하루 수억 건일 때 단일 MySQL 테이블의 한계와 파티셔닝

---

**레포:** https://github.com/serelee08/event-log-pipeline
