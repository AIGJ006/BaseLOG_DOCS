# Feature Specification: 관리자 통계

**Feature Branch**: `024-admin-stats`

**Created**: 2026-10-14

**Status**: Implemented (BaseLOG PR #129 — dev)

**Input**: 민서 요청(2026-10-14) 관리자 화면에 통계 탭 — 방문자, 최근 30일 그래프, 활동 수, 많이 본 글. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-14

- Q: 어디에 보이나? → A: `/admin/stats`. 관리자 메뉴 맨 끝에 **[통계]**. 관리자만 본다(아니면 404).
- Q: 방문자는 어떻게 세나? → A: 화면이 하루 한 번 `POST /api/visits`를 보낸다. 서버는 그날 방문자를 Redis HyperLogLog에 더한다. 누가 왔는지는 남지 않고 수만 센다.
- Q: 같은 사람은? → A: 하루 한 번만 센다. 방문자 구분은 009 조회수와 같다(회원 번호, 비회원은 `vid` 쿠키 해시). `vid`가 없거나 형식이 틀린 비회원에게는 이 요청에서 새 `vid`를 주고 그 값으로 센다.
- Q: 빼는 사람은? → A: 관리자, 봇·링크 미리보기·미리 불러오기(009와 같은 단어 목록). 화면도 관리자면 보내지 않는다.
- Q: DB에는? → A: 5분마다 어제·오늘 수만 `site_visit_daily`에 옮긴다. 이미 있으면 큰 값을 남긴다(줄지 않음). 오늘·어제는 DB 수와 Redis 지금 수 중 큰 값을 보여 준다.
- Q: 많이 본 글은? → A: 최근 7일 조회수가 많은 글 5개. 비회원이 볼 수 있는 전체 공개 글만(공용 `VisibilityFilter`). 친구 공개·비공개 글은 제목이 드러나지 않게 뺀다.
- Q: Redis가 멈추면? → A: 방문 기록은 건너뛰고 화면을 막지 않는다. 옮기기는 그 회차를 건너뛴다. 통계는 DB에 옮겨 둔 수를 쓴다.

## User Scenarios & Testing

### User Story 1 - 방문 기록 (P1)

1. **Given** 비회원이 같은 `vid`로 세 번, 회원이 두 번 방문, **Then** 오늘 방문자 2, 그중 회원 1.
2. **Given** `vid` 쿠키가 없는 비회원, **When** 방문, **Then** 응답에 새 `vid` 쿠키. 다음 방문은 같은 사람으로 센다.
3. **Given** 관리자 또는 `Googlebot`, **When** 방문, **Then** 세지 않는다(응답은 같은 204).
4. **Given** 화면, **Then** 한국 시간 하루에 같은 브라우저·같은 회원은 한 번만 보낸다. 보내기에 실패하면 표시를 남기지 않아 다음 화면에서 다시 보낸다.

### User Story 2 - 통계 보기 (P1)

1. **When** 관리자가 [통계], **Then** "2026년 10월 14일 기준", 방문자 칸 4개(오늘·어제·최근 7일·누적, 누적에는 "10/14부터"), 안내 "같은 사람은 하루 한 번만 세요. 관리자와 검색 로봇은 세지 않고, 5분마다 모아요."
2. **When** 최근 30일 그래프, **Then** 하루 막대 하나. [방문자]·[글 조회수]로 바꾼다. 막대에 마우스를 올리거나 누르면 그날 값(방문자면 회원 수도)을 위에 보여 준다. 기본은 오늘. "30일 합계"도 함께. 숨긴 표로 화면 읽기 프로그램도 읽는다. 그래프는 공용 `DailyBarChart`(PR #134에서 꺼냄, 028 내 둥지 통계와 함께 씀).
3. **When** 활동 표, **Then** 새 회원·새 글·새 댓글·글 조회수를 오늘·최근 7일·전체로.
4. **When** 많이 본 글, **Then** 순위·제목(글로 이동)·@주소·"N회". 없으면 "아직 조회된 글이 없어요". 아래 "전체 공개 글만 보여요."
5. [새로고침]으로 다시 불러온다. 실패하면 "불러오지 못했어요" + [다시 불러오기].

### User Story 3 - 권한 (P1)

1. 비로그인 → 401, 일반 회원 → 404.

## Requirements

- **FR-001** `POST /api/visits`는 비회원도 부른다. 셌든 뺐든 건너뛰었든 `204`(본문 없음, `no-store`).
- **FR-002** 같은 방문자 1분 30번 초과는 429(`ratelimit:visit:{방문자 키}`). Redis 장애면 통과.
- **FR-003** 방문 기록은 어떤 경우에도 화면을 막지 않는다. 트랜잭션을 열지 않는다.
- **FR-004** 옮기기는 `blog.stats.visit-flush-interval`(기본 5m)마다, ShedLock `visit-flush`. 어제도 다시 본다(자정 직전 방문을 놓치지 않으려고).
- **FR-005** `GET /api/admin/stats`는 관리자만(014 `ModerationAccess.requireAdmin`). 응답 `no-store`.
- **FR-006** 날짜는 서비스 시간대(Asia/Seoul). 일별은 오늘 포함 30일, 오래된 날부터, 없는 날은 0.
- **FR-007** 누적 방문자 = 30일 전까지 DB 합계 + 최근 30일 합계. `since`는 처음 센 날(없으면 null).
- **FR-008** 회원 수는 탈퇴 제외, 가입 시각 기준. 글 수는 등록한 글(지운 글·숨긴 글 제외, 공개 범위 무관), 처음 등록 시각 기준. 댓글 수는 지운·숨긴 댓글 제외.
- **FR-009** 일별 조회수는 `post_view_daily` 합계, 누적 조회수는 `post.view_count` 합계(지운 글 제외).
- **FR-010** 원칙 II 읽기 예외: `StatsQueryRepository`(028에서는 `MyStatsQueryRepository`도)가 `member`·`post`·`comment`·`post_view_daily`를 읽기 전용으로 합계만 센다(012 `TrendingRepository`와 같은 방식). `site_visit_daily`는 stats 모듈만 읽고 쓴다.

## 데이터

- **§1 테이블** `V13__site_visit_daily.sql` — `site_visit_daily(visit_date date PK, visitors integer, member_visitors integer, updated_at timestamptz)`, `visitors >= 0 AND member_visitors >= 0` 검사. 누가 왔는지는 담지 않는다. 새 테이블 하나만 더해 무중단 배포.
- **§2 Redis** `stats:visit:{yyyyMMdd}` 그날 방문자, `stats:visit:{yyyyMMdd}:member` 그중 로그인 회원. 둘 다 HyperLogLog(`PFADD`/`PFCOUNT`), TTL 3일.
- **§3 화면 저장** `localStorage` `baselog.visit` = `{한국 날짜}:{m회원번호|guest}`.
- **§4 설정** `blog.stats.visit-flush-interval: 5m`. 일별 30일, 많이 본 글 5개는 코드 상수.

## API

`POST /api/visits` (CSRF) → `204`, `vid`가 없으면 `Set-Cookie: vid`(009와 같은 속성).

`GET /api/admin/stats` (로그인, 관리자) → 200 (`Cache-Control: no-store`)

```json
{"today":"2026-10-14",
 "visitors":{"today":1,"yesterday":5,"last7Days":6,"last30Days":6,"total":13,"since":"2026-09-04"},
 "daily":[{"date":"2026-09-15","visitors":0,"memberVisitors":0,"views":0}],
 "members":{"total":2,"today":2,"last7Days":2},
 "posts":{"total":2,"today":2,"last7Days":2},"comments":{"total":0,"today":0,"last7Days":0},
 "views":{"today":13,"last7Days":13,"total":13},
 "topPosts":[{"postId":1,"title":"...","handle":"writer01","views":9}]}
```

| 상태 | 경우 |
|---|---|
| 401 | 비로그인 |
| 404 | 관리자 아님 |
| 429 | 방문 기록 1분 30번 초과 |

## 시험

- 백엔드: `StatsApiIT`(하루 한 번만 셈·쿠키 없는 비회원에게 vid·관리자와 봇 제외·옮긴 수가 DB에 남고 누적에 더해짐·회원·글·조회수와 많이 본 글(비공개 글 제외)·일반 회원 404, 비로그인 401). `FlywayBaselineIntegrationTest`에 V13.
- 화면: `AdminStatsPage.test.tsx`(방문자 칸·활동 표·많이 본 글, 그래프를 글 조회수로 바꾸면 합계도 바뀜, 많이 본 글 없음 안내), `SiteVisitBeacon.test.tsx`(하루 한 번, 실패하면 다시, 관리자는 보내지 않음).
