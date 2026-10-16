# Feature Specification: 고정 글

**Feature Branch**: `029-pinned-post`

**Created**: 2026-10-16

**Status**: Implemented (BaseLOG PR #168 — dev)

**Input**: 민서 요청(2026-10-16) "고정글 추가". 처음 설계의 나민서 개인 확장 후보 "고정 글"([docs/01-common-requirements.md](../../docs/01-common-requirements.md) §2-4, [docs/03-erd.md](../../docs/03-erd.md) `post.pinned_at`, [docs/10-post-list.md](../../docs/10-post-list.md) "개인 확장")을 만든다. 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-16

- Q: 누가 고정하나? → A: 글쓴이가 자기 글만. 관리자 예외 없음.
- Q: 어디서 고정하나? → A: 내 둥지 관리 > [글]의 등록 글 줄 [고정]·[고정 해제](028 화면). 글 화면에는 단추를 두지 않는다.
- Q: 어디에 보이나? → A: 둥지 첫 목록(검색·태그·카테고리 필터 없음) 맨 앞에 "📌 고정" 표시와 함께. 아래 일반 목록에는 같은 글이 다시 나오지 않는다. 필터·검색 목록과 홈·피드는 고정을 따지지 않는다.
- Q: 몇 개까지? 순서는? → A: 한 둥지에 최대 3개. 여럿이면 최근에 고정한 글이 먼저.
- Q: 비공개·친구 공개 글은? → A: 고정할 수 있다. 보이는지는 기존 공개 범위 규칙 그대로(비공개 글은 주인이 봐도 둥지 목록에 없음, 친구 공개 글은 주인·친구만).
- Q: 임시글은? → A: 고정할 수 없다(409).
- Q: 휴지통으로 옮기면? → A: 고정이 풀린다. 복구해도 다시 고정해야 한다.

## User Scenarios & Testing

### User Story 1 - 고정·해제 (P1)

1. **Given** 내 둥지 관리 > [글] > 등록 글, **When** 줄의 [고정], **Then** 제목 앞에 "📌 고정" 배지, 단추가 [고정 해제](`aria-pressed=true`), 알림 "둥지 맨 위에 고정했어요".
2. **Given** 고정한 글, **When** [고정 해제], **Then** 배지가 사라지고 알림 "고정을 풀었어요".
3. **Given** 이미 고정 글 3개, **When** 다른 글 [고정], **Then** 그 줄 아래 "고정 글은 3개까지예요. 다른 글의 고정을 풀어 주세요", 고정되지 않는다.

### User Story 2 - 둥지에서 보기 (P1)

1. **Given** 고정 글이 있는 둥지, **When** 첫 화면, **Then** 카드 목록 맨 앞에 고정 글(최근 고정한 순), 썸네일 오른쪽 위 "📌 고정"(친구 공개 표시는 왼쪽 위 그대로). 그 뒤로 고정 안 한 글이 최근 순.
2. **Given** 고정 글이 있는 둥지, **When** 태그·카테고리 필터를 고르거나 둥지 안에서 검색, **Then** 고정 칸이 없고 고정 글도 제자리에 섞인다.
3. **Given** 고정 글만 있고 다른 글이 없는 둥지, **When** 첫 화면, **Then** 고정 글이 보이고 빈 둥지 문구는 보이지 않는다.
4. **Given** 고정 글 조회가 실패(404 말고), **When** 첫 화면, **Then** 일반 목록은 그대로 보이고, 이전 고정 결과는 지우며, 일반 글이 없어도 빈 둥지 문구는 띄우지 않는다. 404는 고정 글이 없는 것으로 본다.

## Requirements

- **FR-001** `PUT /api/posts/{postId}/pin` 고정, `DELETE /api/posts/{postId}/pin` 해제. 판정 순서는 004 공개 범위 변경과 같다: 비회원 401 → 계정 상태 403 → 내 휴지통 밖 글이 아니면 404 → 임시글 409 `PIN_NOT_PUBLISHED` → 이미 3개면 409 `PIN_LIMIT_EXCEEDED`.
- **FR-002** 이미 고정한 글 고정·고정 안 한 글 해제는 바꾸지 않고 지금 상태를 돌려준다.
- **FR-003** 개수 확인은 회원별 advisory lock(`pg_advisory_xact_lock(29, 회원)`)으로 한 번에 하나씩 — 두 글을 동시에 고정해도 3개를 넘지 않는다. 개수는 등록·휴지통 밖 고정 글만 센다.
- **FR-004** `GET /api/members/{handle}/posts/pinned`: 고정 글 카드, `pinned_at` 최신 순, 최대 3개, 공개 범위 조건은 둥지 목록과 같다. `nextCursor`는 항상 `null`.
- **FR-005** 필터 없는 둥지 목록(`GET /api/members/{handle}/posts`)은 고정 글을 뺀다(`pinned_at IS NULL`). 태그·카테고리 목록은 빼지 않는다.
- **FR-006** 휴지통으로 옮길 때 `pinned_at`을 지운다.
- **FR-007** 카드(`PostCard`)와 관리 목록(`ManagePostItem`)에 `pinned`(불리언). 화면은 둥지 고정 칸에서만 카드 표시를 단다.
- **FR-008** 3개는 `PostPinService.MAX_PINNED` 코드 상수.

## Success Criteria

- **SC-001** 한 둥지의 등록·휴지통 밖 고정 글은 동시 요청에서도 3개를 넘지 않는다(4번째는 409 `PIN_LIMIT_EXCEEDED`).
- **SC-002** 필터 없는 둥지 첫 화면에서 고정 글은 최근 고정 순으로 맨 앞에 한 번만 보이고, 일반 목록에는 나오지 않는다.
- **SC-003** 태그·카테고리·검색 목록과 홈·피드의 순서는 고정 전과 같다.
- **SC-004** 오류 응답은 FR-001 판정 순서와 아래 API 표의 상태·코드를 따른다.

## Assumptions

- 공개 범위는 기존 규칙 그대로다. 고정해도 볼 수 없는 사람에게는 보이지 않는다.
- 임시글은 고정 대상이 아니다. 등록 글만 고정한다.
- 휴지통으로 옮기면 고정이 풀리고, 복구해도 자동으로 다시 고정되지 않는다.

## 데이터

- `V21__post_pinned.sql`: `post.pinned_at timestamptz NULL` 칸 추가만(값 없음 = 고정 안 함). 인덱스는 두지 않는다 — 한 회원의 글만 보므로 기존 `ix_post_blog`로 충분하고, 배포 중 `post` 쓰기를 막지 않는다.

## API

`PUT /api/posts/{postId}/pin` · `DELETE /api/posts/{postId}/pin` (로그인, CSRF) → 200 (`Cache-Control: no-store`)

```json
{"pinned": true, "pinnedAt": "2026-10-16T05:00:00.123456Z"}
```

| 상태 | 코드 | 경우 |
|---|---|---|
| 401 | `LOGIN_REQUIRED` | 비회원 |
| 403 | `EMAIL_NOT_VERIFIED` 등 | 계정 상태 |
| 404 | `NOT_FOUND` | 없음·남의 글·휴지통 글 |
| 409 | `PIN_NOT_PUBLISHED` | 임시글 ("등록한 글만 고정할 수 있어요") |
| 409 | `PIN_LIMIT_EXCEEDED` | 이미 3개 ("고정 글은 3개까지예요. 다른 글의 고정을 풀어 주세요") |

`GET /api/members/{handle}/posts/pinned` → 200 `PostCardPage` (`items[].pinned = true`, `nextCursor = null`), 없는 둥지 404.

## 시험

- 백엔드: `PostPinIT`(고정 칸 순서·첫 목록에서 빠짐·해제 뒤 제자리, 같은 요청 두 번·4번째 409, 임시글 409·남의 글/없는 글 404·비회원 401, 비공개 고정 글은 안 보임·휴지통이면 해제, 관리 목록 `pinned`), `FlywayBaselineIntegrationTest`(V21 칸 1개).
- 화면: `PublishedRow.test.tsx`([고정]→배지·[고정 해제]·알림, 3개 초과 문구), `BlogPage.test.tsx`(고정 글이 맨 앞·표시 1개, 고정 글만 있으면 빈 문구 없음, 태그 필터 중엔 부르지 않음).
