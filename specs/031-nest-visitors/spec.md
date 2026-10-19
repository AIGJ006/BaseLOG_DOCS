# Feature Specification: 둥지별 방문자

**Feature Branch**: `031-nest-visitors`

**Created**: 2026-10-19

**Status**: Implemented (BaseLOG PR #173 — dev)

**Input**: 민서 요청(2026-10-19) "둥지별 통계 구현해줘". 028 내 둥지 관리 [통계]에 내 둥지 방문자 수를 더한다(028에서 "둥지별로는 세지 않는다"로 뺐던 것). 방법은 024 사이트 방문자와 같다. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-19

- Q: 무엇을 세나? → A: 둥지(`/@주소`)마다 그날 들어온 사람 수. 누가 왔는지는 남기지 않고 수만 센다.
- Q: 어떻게 세나? → A: 024와 같다. 화면이 둥지를 열면 그 둥지에 하루 한 번 `POST /api/members/{handle}/visits`를 보낸다. 서버는 둥지마다 그날 방문자를 Redis HyperLogLog(`stats:nest-visit:{yyyyMMdd}:{주인 번호}`)에 더하고, 5분마다 그 수를 `blog_visit_daily`(V24)에 옮긴다(큰 값을 남김).
- Q: 같은 사람은? → A: 둥지마다 하루 한 번만 센다. 방문자 구분은 009 조회수와 같다(회원 번호, 비회원은 `vid` 쿠키 해시). `vid`가 없거나 틀린 비회원에게는 이 요청에서 새 `vid`를 주고 그 값으로 센다.
- Q: 누구를 빼나? → A: 둥지 주인 본인, 관리자, 봇·링크 미리보기·미리 불러오기(024와 같은 목록). 화면도 내 둥지와 관리자는 보내지 않는다.
- Q: 어디에 보이나? → A: 내 둥지 관리 [통계] 맨 위 **둥지 방문자** 칸 4개(오늘·최근 7일·최근 30일·누적)와 최근 30일 그래프의 [방문자] 항목(기본 선택). 안내 "내 둥지에 들어온 사람 수예요. 같은 사람은 하루 한 번만 세고, 나와 검색 로봇은 세지 않아요."
- Q: 남의 둥지 방문자 수도 보이나? → A: 아니다. 본인 통계에만 있다. 관리자 통계(024)는 사이트 전체 방문자 그대로다.
- Q: 언제부터 세나? → A: 배포한 날부터. 이전 방문은 없다(0).

## User Scenarios & Testing

### User Story 1 - 둥지 방문 세기 (P1)

1. **Given** 비회원이 같은 `vid`로 A 둥지를 세 번, 회원이 A 둥지를 두 번 열면, **Then** A 둥지의 오늘 방문자는 2.
2. **Given** 같은 사람이 A·B 둥지를 열면, **Then** A·B 둥지에 1씩 더해진다.
3. **Given** 둥지 주인 본인·관리자·봇이 열면, **Then** 세지 않고 204.
4. **Given** 없는 둥지 주소, **Then** 204로 조용히 넘어간다(있는지 없는지 알려 주지 않는다).
5. **Given** 같은 방문자가 1분에 60번 넘게 보내면, **Then** 429.

### User Story 2 - 내 둥지 방문자 보기 (P1)

1. **When** 내 둥지 관리 [통계], **Then** 맨 위 "둥지 방문자" 칸 4개(오늘·최근 7일·최근 30일·누적)와 안내, 그 아래 기존 글 조회수 칸.
2. **When** 최근 30일 그래프, **Then** [방문자]·[조회수]·[받은 좋아요]·[받은 댓글]로 바꾸고, 기본은 [방문자].
3. **Given** 5분 옮기기 전, **Then** 어제·오늘은 Redis의 지금 수와 DB 중 큰 쪽을 보여 준다.

## Requirements

- **FR-001** `POST /api/members/{handle}/visits` — 비회원도 부를 수 있고 CSRF는 001 공통. 셌든 뺐든 건너뛰었든 항상 `204`(본문 없음, `Cache-Control: no-store`). 같은 방문자 1분 60번 초과는 429(`ratelimit:nest-visit:{방문자 키}`, 여러 둥지를 돌아다녀도 막히지 않게 사이트 방문 30번보다 넉넉히). Redis 장애면 세지 않고 통과.
- **FR-002** 판정 순서: 관리자·봇이면 건너뜀 → 둥지 주인(001 `findReadableBlogOwner`)이 없거나 탈퇴 중이면 건너뜀 → 주인 본인이면 건너뜀 → 방문자 키 → 요청 제한 → Redis 더하기.
- **FR-003** 화면 `NestVisitBeacon`은 둥지마다 하루 한 번(한국 시간 날짜, 같은 브라우저·같은 회원) 보낸다. 보낸 표시는 `localStorage`의 `baselog.nestVisits`에 오늘 것만 남긴다. 실패하면 다음에 또 보낸다. 화면에는 아무것도 그리지 않는다.
- **FR-004** 옮기기는 024 `VisitFlushJob`이 같은 회차(기본 5분, ShedLock `visit-flush`)에 그제·어제·오늘을 다시 본다. 값은 줄지 않는다.
- **FR-005** `GET /api/me/stats` 응답에 `visitors {today, last7Days, last30Days, total}`을 더하고, `days[]`의 하루마다 `visitors`를 더한다. 누적 = DB 합계(어제·오늘은 Redis와 큰 쪽).
- **FR-006** 탈퇴로 회원 행이 지워지면 `blog_visit_daily` 행도 함께 지워진다(FK CASCADE).

## 데이터

- **V24** `blog_visit_daily(owner_id FK → member CASCADE, visit_date, visitors ≥ 0, updated_at)`, PK (owner_id, visit_date). 서비스 시간대(Asia/Seoul) 날짜. 새 테이블 하나만 더해 무중단 배포에 문제없다.
- **Redis** `stats:nest-visit:{yyyyMMdd}:{주인 번호}` HyperLogLog(그날 그 둥지 방문자), `stats:nest-visit:{yyyyMMdd}:owners`(그날 방문자가 있는 둥지 주인 번호, 옮길 때 씀). 둘 다 TTL 3일(024와 같음).

## API

| 요청 | 응답 |
|---|---|
| `POST /api/members/{handle}/visits` | 항상 204, 1분 60번 초과 429. 비회원에게 `vid`가 없으면 `Set-Cookie: vid` |
| `GET /api/me/stats` | 028 응답 + `visitors`, `days[].visitors` |

## 시험

- 백엔드: `BlogVisitApiIT`(둥지마다 하루 한 번, 주인·관리자·봇 제외, 없는 둥지도 204, 쿠키 없는 비회원에게 `vid`, 옮긴 수가 DB에 남고 누적에 더해짐), `FlywayBaselineIntegrationTest`(V24 표·PK·CHECK).
- 화면: `NestVisitBeacon.test.tsx`(하루 한 번, 내 둥지·관리자는 안 보냄, 실패하면 다시), `ManageStatsPage.test.tsx`(방문자 칸·그래프 기본 [방문자]).
