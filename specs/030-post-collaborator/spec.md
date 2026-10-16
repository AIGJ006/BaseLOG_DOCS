# Feature Specification: 공동 작업자

**Feature Branch**: `030-post-collaborator`

**Created**: 2026-10-16

**Status**: Implemented (BaseLOG PR #169 — dev)

**Input**: 민서 요청(2026-10-16) "공동작업자 기능" — 마감 날이라 작은 범위로(코디네이터 추천안, 민서 "ㄱㄱ"). 처음 설계의 나민서 개인 확장 후보 "공동 작업자"([docs/01-common-requirements.md](../../docs/01-common-requirements.md) §2-4 `post_collaborator`)를 만든다. 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

**공통 스펙 예외**: FR-003~FR-005는 [002 FR-036](../002-post-authoring/spec.md)(저장·발행·변경 취소는 작성자만)의 공동 작업자 예외다. 공동 작업자는 등록 글에서만 자동 저장·저장·변경 취소·등록을 할 수 있고, 그 밖의 일(FR-006)은 그대로 작성자만 한다.

## Clarifications

### Session 2026-10-16

- Q: 누구를 고르나? → A: 글쓴이가 자기 친구(수락된 친구) 중에서. 초대·수락 단계는 없다.
- Q: 몇 명까지? → A: 글 한 개에 최대 3명.
- Q: 공동 작업자는 무엇을 할 수 있나? → A: 그 글의 제목·본문 수정과 다시 등록만. 자동 저장·임시저장·변경 취소도 된다. 삭제·공개 범위·태그·카테고리·썸네일·공동 작업자 바꾸기는 글쓴이만.
- Q: 어디서 고르나? → A: 글쓰기·수정 화면의 등록 창(공개 범위·태그를 고르는 곳)에서 친구 목록으로.
- Q: 공동 작업자는 어디로 들어가나? → A: 글 화면의 [수정]으로만. 내 둥지 관리에는 그 글이 보이지 않는다.
- Q: 임시글은? → A: 글쓴이는 임시글에서도 고를 수 있지만, 공동 작업자가 열 수 있는 건 등록한 글뿐이다.
- Q: 친구를 끊으면? → A: 서로의 글에서 공동 작업자가 빠진다(거절·취소 포함).
- Q: 동시에 고치면? → A: 동시 편집은 없다. 기존 버전 확인 그대로 — 늦게 저장한 쪽이 "다른 탭이나 기기에서 이 글이 수정되었어요"를 본다.
- Q: 알림은? → A: 공동 작업자로 더해지면 그 사람에게 한 번("…님이 「글」의 공동 작업자로 추가했어요"). 끌 수 없다. 그 글을 읽을 수 없으면(비공개 등) 만들지 않는다.
- Q: 나만 보기 글은? → A: 공동 작업자가 있는 글은 나만 보기로 할 수 없다(민서 "공동작업자 글은 나만보기가 불가능하게 해서 올리자"). 나만 보기 글에는 공동 작업자를 고를 수 없다. 공동 작업자를 모두 빼면 다시 나만 보기로 할 수 있다.

## User Scenarios & Testing

### User Story 1 - 공동 작업자 고르기 (P1)

1. **Given** 글쓴이의 등록 창, **When** "공동 작업자"에서 친구를 체크하고 [글 등록], **Then** 등록이 끝난 뒤 고른 사람이 저장되고 그 사람에게 알림이 간다.
2. **Given** 이미 3명을 고름, **When** 등록 창을 열면, **Then** 다른 친구 체크 칸은 막힌다.
3. **Given** 공동 작업자를 고름, **When** 공개 범위를 열면, **Then** 공개 범위의 "나만 보기"는 고를 수 없다.
4. **Given** 친구가 없음, **When** 등록 창을 열면, **Then** 안내 "친구를 맺으면 공동 작업자로 고를 수 있어요".

### User Story 2 - 공동 작업자가 고치기 (P1)

1. **Given** 내가 공동 작업자인 글, **When** 글 화면을 열면, **Then** [수정] 단추만 보인다([공개 범위]·[삭제] 없음).
2. **Given** 공동 작업자, **When** [수정]으로 에디터를 열면, **Then** 파일·사진 올리기, 공동 작업자 고르기, 등록 창의 공개 범위·태그·카테고리·썸네일이 없다.
3. **Given** 공동 작업자가 제목·본문을 고침, **When** [글 등록]을 누르면, **Then** 제목·본문만 바뀌고 공개 범위·태그·썸네일·글쓴이는 그대로다.

### User Story 3 - 함께 쓴 사람 보기 (P2)

1. **Given** 공동 작업자가 있는 글, **When** 누구든 글 화면, **Then** 글쓴이 정보 근처에 "함께 쓴 사람"과 작은 프로필 사진·닉네임(누르면 그 사람 둥지).

## Requirements

- **FR-001** `GET /api/posts/{postId}/collaborators`·`PUT /api/posts/{postId}/collaborators`(`{"handles": [...]}`, 통째로 바꿈, 빈 목록이면 모두 뺌): 글쓴이만(내 휴지통 밖 글, 임시글 포함). 판정 순서 401 → 403 계정 상태 → 404 내 글 아님 → 400.
- **FR-002** 400 `VALIDATION_FAILED`, `errors[].field = "handles"`: 3명 초과 `TOO_MANY_COLLABORATORS`("공동 작업자는 3명까지 고를 수 있어요"), 친구 아님·나 자신·없는 주소 `COLLABORATOR_NOT_FRIEND`("친구만 공동 작업자로 고를 수 있어요").
- **FR-003** 고칠 수 있는 글 = 내 휴지통 밖 글, 또는 휴지통 밖 등록 글 중 내가 공동 작업자이고 지금도 글쓴이와 친구인 글. 에디터 열기·자동 저장·수동 저장·변경 취소·등록이 이 판정을 쓰고, 아니면 404(구분 없음). 공동 작업자의 등록은 글 행을 잠근 뒤 한 번 더 판정한다(그 사이 빠졌으면 404).
- **FR-004** 공동 작업자가 저장해도 Redis 보관분 주인·행 잠금·사진 연결은 글쓴이 기준 — 글쓴이와 공동 작업자가 같은 작업본을 이어서 본다.
- **FR-005** 공동 작업자의 등록은 요청의 공개 범위·태그·썸네일을 버리고 글에 있는 값을 다시 쓴다. 등록 이벤트(`PostEdited`)의 회원은 글쓴이.
- **FR-006** 삭제·복구·영구 삭제·공개 범위·카테고리·고정·공동 작업자 바꾸기 API는 그대로 글쓴이만(공동 작업자는 404).
- **FR-007** 에디터 응답(`WorkingCopy`)에 `collaborator`(불리언 — 연 사람이 공동 작업자인가).
- **FR-008** 글 상세(`PostDetail`)에 `collaborators: [{handle, nickname, profileImageUrl}]`(고른 순, 친구가 아니게 됐거나 탈퇴한 회원은 뺌)와 `viewer.isCollaborator`.
- **FR-009** 친구 끊기·거절·취소(`FriendshipEnded` 이벤트, 같은 트랜잭션)에서 두 사람 사이의 공동 작업자 행을 모두 지운다. 공동 작업자를 더할 때는 두 사람의 친구 행을 `FOR SHARE`로 잠근 채 친구인지 보고 넣는다 — 끊기가 먼저 끝나면 400, 더하기가 먼저면 끊기가 기다렸다가 새 행까지 지운다. 권한 확인과 목록도 지금 친구인지 함께 본다.
- **FR-010** 알림 종류 `COLLABORATOR_ADDED`(한 건짜리, 행동자 = 글쓴이, 글 링크). 새로 더해진 사람에게만, 끌 수 없음, 받는 사람이 그 글을 읽을 수 없으면 만들지 않는다.
- **FR-011** 공동 작업자가 있는 글은 나만 보기로 바꿀 수 없다 — 등록(`PUBLISH` 요청의 `visibility`)·공개 범위 바꾸기 모두 400 `VALIDATION_FAILED`, `errors[].field = "visibility"`, `COLLABORATOR_POST_PRIVATE`("공동 작업자가 있는 글은 나만 보기로 할 수 없어요"). 나만 보기 글에 공동 작업자를 고르면 400, `field = "handles"`, `PRIVATE_POST_COLLABORATORS`("나만 보기 글에는 공동 작업자를 고를 수 없어요"). 두 확인 모두 글 행을 잠근 뒤에 한다.

## Success Criteria

- **SC-001** 공동 작업자가 아닌 회원(친구여도)은 남의 글의 에디터·저장·등록·변경 취소에서 모두 404를 받는다.
- **SC-002** 공동 작업자는 삭제·공개 범위·카테고리 변경에서 404를 받고 글은 그대로다.
- **SC-003** 공동 작업자가 다시 등록해도 공개 범위·태그·글쓴이가 바뀌지 않는다.
- **SC-004** 친구를 끊으면 바로 그 글을 열 수 없다.
- **SC-005** 공동 작업자가 있는 글은 어떤 경로로도 나만 보기가 되지 않는다.

## Assumptions

- 공동 작업자가 올린 사진은 글쓴이 사진이 아니라서 글에 붙지 않는다 — 그래서 공동 작업자 에디터에는 파일·사진 올리기가 없다.
- 공동 작업자는 내 둥지 관리·통계에 그 글이 나오지 않는다. 글의 조회수·좋아요는 글쓴이 것이다.

## 데이터

- `V22__post_collaborator.sql`: 새 표 `post_collaborator(post_id, member_id, created_at)`, PK(post_id, member_id), 글·회원 FK `ON DELETE CASCADE`, `ix_post_collaborator_member`. 알림 종류 CHECK에 `COLLABORATOR_ADDED`를 더한다(끌 수 있는 종류 CHECK는 그대로). 기존 표·칸은 바꾸지 않는다. 새 CHECK는 `NOT VALID`로 건다.
- `V23__validate_notification_type.sql`: V22의 CHECK를 `VALIDATE`한다(표 전체를 오래 잠그지 않도록 나눔).

## API

`GET·PUT /api/posts/{postId}/collaborators` → 200

```json
{"items": [{"handle": "minseo", "nickname": "민서", "profileImageUrl": null}]}
```

| 상태 | 코드 | 경우 |
|---|---|---|
| 401 | `LOGIN_REQUIRED` | 비회원 |
| 403 | `EMAIL_NOT_VERIFIED` 등 | 계정 상태 (PUT) |
| 404 | `NOT_FOUND` | 없음·남의 글(공동 작업자 포함)·휴지통 글 |
| 400 | `VALIDATION_FAILED` + `TOO_MANY_COLLABORATORS` / `COLLABORATOR_NOT_FRIEND` / `PRIVATE_POST_COLLABORATORS` | FR-002·FR-011 |

## 시험

- 백엔드: `PostCollaboratorIT`(고르기·목록·글 상세·알림 한 번, 친구 아님·나 자신·4명 400·남의 글 404, 공동 작업자 저장·등록에서 공개 범위·태그 그대로, 공동 작업자 삭제·공개 범위·카테고리 404, 남·친구의 에디터·저장·등록·변경 취소 404, 임시글은 못 엶, 친구 끊으면 빠짐, 공동 작업자 있는 글은 나만 보기 400·모두 빼면 됨, 나만 보기 글에 고르기 400), `NotificationTypeTest`, `FlywayBaselineIntegrationTest`(V22 표 1·칸 3·FK 2·인덱스 1).
- 화면: `PublishDialog.collaborators`(3명이면 칸이 꺼짐·친구 없음 안내·목록 읽기 실패 시 [다시 시도]·[더 보기]·등록 성공 뒤에만 저장·안 바꿨으면 저장 안 함·저장 400 문구·고르면 나만 보기 막힘·나만 보기 글이면 칸 꺼짐), `EditorPage.collaborator`(공동 작업자 에디터에 사진·첨부·카테고리 없음, 등록 창은 다시 등록만 묻고 작업본 값 그대로, [임시저장] 뒤 글로 돌아감), `PostDetailPage.author`(함께 쓴 사람, 공동 작업자는 [수정]만), `VisibilitySelect`(끈 항목·안내, 400 칸별 문구), `notificationText`(COLLABORATOR_ADDED).
