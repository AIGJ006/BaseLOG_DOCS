# Feature Specification: 내 둥지 관리

**Feature Branch**: `028-nest-manage`

**Created**: 2026-10-15

**Status**: Implemented (BaseLOG PR #134 — dev)

**Input**: 민서 요청(2026-10-15) [내 글 관리]를 [내 둥지 관리]로 바꾸고, 글·카테고리·휴지통을 왼쪽 탭 한 화면에 모으고, 내 글 통계 탭을 더한다. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-15

- Q: 무엇이 바뀌나? → A: 006 내 글 관리(`/manage/posts`)와 017 카테고리 관리(`/manage/categories`)가 따로 있던 화면을 **내 둥지 관리** 한 화면으로 묶는다. 설정 화면처럼 왼쪽 세로 탭 **[글] [카테고리] [통계] [휴지통]**, 오른쪽에 고른 화면. 계정 메뉴 이름도 [내 글 관리] → [내 둥지 관리].
- Q: 주소는? → A: 탭마다 주소가 있다(`/manage/posts`·`/manage/categories`·`/manage/stats`·`/manage/trash`). 뒤로 가기·새로 고침에도 그대로. `/manage`는 `/manage/posts`로 보낸다.
- Q: 옛 주소는? → A: `/manage/posts`·`/manage/categories`는 그대로 쓴다. 휴지통이 왼쪽 탭으로 나가서 옛 `/manage/posts?tab=trash`는 `/manage/trash`로 보낸다(replace). [글] 안의 탭은 [등록 글]·[임시글] 둘만 남는다.
- Q: 통계에는 무엇이? → A: 내 글 조회수(오늘·최근 7일·최근 30일·누적), 최근 30일 그래프(조회수 / 받은 좋아요 / 받은 댓글), 활동 표(등록한 글·받은 좋아요·받은 댓글을 오늘·최근 7일·전체로), 최근 30일 많이 본 내 글 5개.
- Q: 둥지 방문자 수는? → A: 넣지 않는다. 방문자는 사이트 전체로만 세고(024) 둥지별로는 세지 않는다.
- Q: 새 테이블은? → A: 없다. 있는 `post`·`post_like`·`comment`·`post_view_daily`를 읽기만 한다.
- Q: 무엇을 빼고 세나? → A: 지운 글은 뺀다. 받은 좋아요·댓글은 **다른 사람이 남긴 것만**(내가 누른 좋아요·내가 쓴 댓글 제외). 댓글은 지운·숨긴 것을 빼고 대댓글은 넣는다.
- Q: 많이 본 글에 비공개 글도? → A: 넣는다. 본인만 보는 화면이라 공개 범위와 상관없이 보여 주고 자물쇠·친구 표시를 붙인다. 숨긴 글은 뺀다.
- Q: 그래프는? → A: 024 관리자 통계의 막대 그래프를 `DailyBarChart`로 꺼내 두 화면이 함께 쓴다.

## User Scenarios & Testing

### User Story 1 - 한 화면에서 관리 (P1)

1. **Given** 로그인, **When** 계정 메뉴 [내 둥지 관리], **Then** `/manage/posts`, 제목 "내 둥지 관리", 안내 "글과 카테고리, 통계, 휴지통을 한곳에서 관리해요.", 왼쪽 탭 [글]·[카테고리]·[통계]·[휴지통 N]. [글]이 켜져 있다(`aria-current="page"`).
2. **Given** [카테고리]나 [통계]로 바로 들어옴, **Then** 휴지통 수를 한 번 따로 물어 [휴지통 N]에 붙인다. [글]·[휴지통]은 목록을 부를 때 수를 알려 주므로 따로 묻지 않는다.
3. **Given** 옛 주소 `/manage/posts?tab=trash`, **Then** `/manage/trash`로 가고 제목 "휴지통". 휴지통에는 [새 글]과 [등록 글]·[임시글] 탭이 없다.
4. **Given** 글 상세에서 [삭제] → 휴지통으로, **Then** `/manage/trash`로 간다(빈 임시글이라 바로 지웠으면 전처럼 `/manage/posts?tab=drafts`).
5. **Given** 폭 640px 이하, **Then** 탭이 위쪽 가로 한 줄이 되고 그림을 빼고 글자만.

### User Story 2 - 내 통계 보기 (P1)

1. **When** [통계], **Then** "2026년 10월 15일 기준", [새로고침], 글 조회수 칸 4개(오늘·최근 7일·최근 30일·누적), 안내 "글 화면의 조회수와 같아요. 같은 사람은 하루 한 번만 세고, 검색 로봇은 세지 않아요."
2. **When** 최근 30일 그래프, **Then** 하루 막대 하나. [조회수]·[받은 좋아요]·[받은 댓글]로 바꾼다(`aria-pressed`). 막대에 마우스를 올리거나 누르면 그날 값을 위에 보인다. 기본은 오늘. "30일 합계"도 함께. 숨긴 표로 화면 읽기 프로그램도 읽는다.
3. **When** 활동 표, **Then** 등록한 글·받은 좋아요·받은 댓글을 오늘·최근 7일·전체로. 아래 "내가 누른 좋아요와 내가 쓴 댓글은 세지 않아요."
4. **When** 최근 30일 많이 본 글, **Then** 순위·공개 범위 표시·제목(내 글로 이동)·"N회". 없으면 "아직 조회된 글이 없어요".
5. 불러오기에 실패하면 "통계를 불러오지 못했어요" + [다시 시도].

### User Story 3 - 권한 (P1)

1. 비로그인 → `GET /api/me/stats` 401. 남의 통계를 가리키는 값은 주소에 없다(006 FR-002와 같음).

## Requirements

- **FR-001** `/manage`는 `ManageNestLayout` 아래 `posts`·`categories`·`stats`·`trash` 네 화면. 탭은 주소가 바뀌는 링크(`NavLink`).
- **FR-002** `/manage/posts?tab=trash`는 `/manage/trash`로 보낸다. `/manage/posts`의 탭은 등록 글·임시글(`ManageTabs`).
- **FR-003** 카테고리 화면 내용은 017 그대로. 제목만 "카테고리"(h2)로, [내 글 관리] 링크는 뺀다. 글 화면의 [카테고리 관리] 링크도 뺀다.
- **FR-004** `GET /api/me/stats`는 로그인 본인 것만(`@LoginRequired`). 응답 `no-store`.
- **FR-005** 날짜는 서비스 시간대(Asia/Seoul). 일별은 오늘 포함 30일, 오래된 날부터, 없는 날은 0.
- **FR-006** 조회수: 오늘·7일·30일은 `post_view_daily` 합계, 누적은 `post.view_count` 합계. 지운 글 제외. 기준은 009 조회수와 같다.
- **FR-007** 글 수는 등록한 글(`PUBLISHED`, 지운 글·숨긴 글 제외, 공개 범위 무관), 처음 등록 시각(`published_at`) 기준. 좋아요·댓글은 받은 시각 기준.
- **FR-008** 많이 본 글은 최근 30일 `post_view_daily` 합계 순(같으면 글 번호 큰 것 먼저), 최대 5개, 숨긴 글 제외, 공개 범위 무관.
- **FR-009** 원칙 II 읽기 예외: `MyStatsQueryRepository`가 `post`·`post_like`·`comment`·`post_view_daily`를 읽기 전용으로 합계만 센다(024 `StatsQueryRepository`와 같은 방식). 누가 봤는지·눌렀는지는 돌려주지 않는다.

## 데이터

- 새 테이블·마이그레이션 없음. 30일·5개는 `MyStatsService` 코드 상수(`DAYS`·`TOP_POSTS`).

## API

`GET /api/me/stats` (로그인) → 200 (`Cache-Control: no-store`)

```json
{"today":"2026-10-15",
 "views":{"today":58,"last7Days":350,"last30Days":870,"total":4321},
 "daily":[{"date":"2026-09-16","views":0,"likes":0,"comments":0}],
 "posts":{"total":24,"today":1,"last7Days":3},
 "likes":{"total":120,"today":2,"last7Days":9},
 "comments":{"total":45,"today":0,"last7Days":5},
 "topPosts":[{"postId":7,"title":"첫 글","visibility":"PUBLIC","views":42}]}
```

| 상태 | 경우 |
|---|---|
| 401 | 비로그인 |

## 시험

- 백엔드: `MyStatsApiIT`(내 글만 세고 내가 누른 좋아요와 내 댓글은 뺀다 — 남의 글 조회수 제외, 일별 30개, 비공개 글도 많이 본 글에; 비로그인 401).
- 화면: `ManageStatsPage.test.tsx`(조회수 칸·활동 표·많이 본 글, 그래프 항목 바꾸기, 조회된 글 없음 안내·다시 시도), `ManagePostsPage.test.tsx`(왼쪽 탭 글·카테고리·통계·휴지통, 옛 `?tab=trash` → 휴지통, 카테고리로 바로 들어와도 휴지통 수), `SiteHeader.test.tsx`(계정 메뉴 내 둥지 관리), `DetailDeleteButton.test.tsx`(휴지통으로 옮기고 내 둥지 관리의 휴지통으로). E2E `manage-posts.spec.ts`(내 둥지 관리 글·휴지통).
