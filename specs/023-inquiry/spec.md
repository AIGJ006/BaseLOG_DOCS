# Feature Specification: 문의·신고

**Feature Branch**: `023-inquiry`

**Created**: 2026-10-14

**Status**: Implemented (BaseLOG PR #121 — dev; 목록·탭 PR #137)

**Input**: 민서 요청(2026-10-14) — 회원이 버그·계정 문의·사용자 신고·기타를 관리자에게 보내고, 관리자가 답하면 🔔 알림으로 알려 준다. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-14

- Q: 어디서 여나? → A: 계정 메뉴의 **[문의·신고]** → `/support`. 위는 문의 보내기, 아래는 "내가 보낸 문의"(2026-10-15부터 두 탭 — 아래 Session 2026-10-15). 로그인 회원만(비로그인은 `/login?returnTo=…`).
- Q: 종류는? → A: 버그 신고(`BUG`) · 계정 문의(`ACCOUNT`) · 사용자 신고(`USER_REPORT`) · 기타(`OTHER`). 처음 고른 값은 버그 신고.
- Q: 사용자 신고는 누구를 신고하나? → A: 사용자 신고를 고르면 "신고할 사람 블로그 주소 (선택)" 칸이 나온다. `@kim`·`kim`·`https://…/@kim/posts/1` 모두 `kim`으로 바꿔 저장한다(소문자). 블로그 주소 형식만 본다. 다른 종류면 이 값은 버린다. 글·댓글 신고는 기존 [신고] 버튼을 안내한다.
- Q: 사진은? → A: 한 장까지(선택). 글 사진과 같은 업로드(presign → 올리기 → complete)로 올리고 번호만 함께 보낸다.
- Q: 정지 회원도 보낼 수 있나? → A: 보낼 수 있다. 정지에 대해 물어볼 길이 있어야 한다.
- Q: 답은 어디서? → A: 관리자 화면 위쪽 메뉴의 **[문의]**(`/admin/inquiries`). [접수]·[답변 완료] 탭, 새것부터 20개씩 [더 보기]. (2026-10-15부터 [전체]·[처리 전]·[처리 완료] 한 줄 목록 + 문의 화면 — 아래 Session 2026-10-15)
- Q: 알림은? → A: 처음 답할 때만 `INQUIRY_ANSWERED` 🔔 알림 "문의에 답변이 달렸어요"(누르면 `/support`, 2026-10-15부터 `/support?tab=mine`). 답을 고칠 때는 다시 알리지 않는다. 운영 알림이라 끌 수 없다.
- Q: 남용 막기는? → A: 한 사람이 하루 10건. Redis 장애면 통과.
- Q: 탈퇴하면? → A: 그 회원이 보낸 문의를 모두 지운다.

### Session 2026-10-15 (목록·탭, PR #137)

- Q: `/support`는 어떻게 나누나? → A: 탭 **[문의·신고하기]**(보내기)와 **[내 문의·신고]**(내가 보낸 문의). 처음엔 [문의·신고하기]가 열린다. [내 문의·신고]는 `/support?tab=mine`이고, 목록을 불러오면 탭 이름 뒤에 건수가 붙는다(`내 문의·신고 1`). 왼쪽·오른쪽 방향키로 탭을 옮긴다. 탭을 옮겨도 쓰던 내용은 남는다.
- Q: 답변 알림은 어디로 데려가나? → A: `/support?tab=mine`([내 문의·신고] 탭).
- Q: 관리자 문의 화면은? → A: 릴리스 노트처럼 한 줄 목록. 탭은 **[전체]**(`/admin/inquiries`, 처음 열리는 탭)·**[처리 전]**(`?status=received`)·**[처리 완료]**(`?status=answered`). 한 줄에 상태 배지(처리 전/처리 완료)·종류·내용 첫머리·보낸 회원(닉네임 @주소, 탈퇴면 "탈퇴한 사용자")·날짜. 새것부터 20개씩 [더 보기].
- Q: 답은 어디서 하나? → A: 한 줄을 누르면 `/admin/inquiries/:id` 문의 화면. 보낸 회원·종류·내용·사진과 답변 칸, 위에 [← 문의 목록]. 서버에 `GET /api/admin/inquiries/{id}`를 더했다.
- Q: 상태 이름은? → A: 관리자 화면은 처리 전(`RECEIVED`)·처리 완료(`ANSWERED`). 회원 화면은 그대로 접수·답변 완료.

### Session 2026-10-16 (목록으로 돌아가기, a79e714)

- Q: 문의 화면의 [← 문의 목록]은 어느 탭으로 가나? → A: 목록에서 열었던 상태 탭으로 돌아간다([처리 전]에서 열면 `/admin/inquiries?status=received`). 주소로 바로 열었거나 모르는 값이면 [전체](`/admin/inquiries`).

## User Scenarios & Testing

### User Story 1 - 문의 보내기 (P1)

1. **Given** 로그인 회원, [문의·신고하기] 탭, **When** 종류·내용을 적고 [보내기], **Then** "문의를 보냈어요. 답이 오면 알려 드릴게요", 칸이 비고, [내 문의·신고] 목록 맨 위에 **접수**로 붙는다(탭은 그대로).
2. **Given** 내용이 비었거나 2,000자를 넘으면, **Then** [보내기]가 꺼진다. 글자 수(`n/2000`)는 넘으면 빨갛다.
3. **Given** 사용자 신고, **When** 블로그 주소 칸에 `@Kim`, **Then** `kim`으로 저장되고 카드에 "신고한 사람: @kim".
4. **Given** 사진을 올렸으면, **Then** 미리보기와 [사진 빼기]. 못 올리면 이유를 보여 준다.
5. **Given** 오늘 10건을 다 보냈거나 서버 오류, **Then** "잠시 후 다시 시도해 주세요". 칸 오류는 서버 문구를 그대로 보여 준다.

### User Story 2 - 내가 보낸 문의 보기 (P1)

1. **When** `/support`의 [내 문의·신고] 탭(`?tab=mine`), **Then** 새것부터 최대 50건. 카드마다 종류 · 보낸 날짜 · 상태(접수/답변 완료) · 내용 · 사진 · 답변(답한 일시와 함께). 답한 관리자는 알려 주지 않는다.
2. 보낸 문의가 없으면 "아직 보낸 문의가 없어요". 못 불러오면 "불러오지 못했어요" [다시 불러오기].

### User Story 3 - 관리자 답변 (P1)

1. **Given** 관리자, **When** [문의], **Then** [전체] 탭 한 줄 목록(상태·종류·내용 첫머리·보낸 회원·날짜). [처리 전]·[처리 완료] 탭은 그 상태만 다시 부른다.
2. **When** 한 줄을 누르면, **Then** `/admin/inquiries/:id`에 보낸 회원(닉네임 @주소 → 회원 화면, 탈퇴면 "탈퇴한 사용자")·종류·내용·사진과 답변 칸. 없는 문의면 "없어진 문의예요", 못 불러오면 "불러오지 못했어요" [다시 불러오기]. 위의 [← 문의 목록]은 열었던 상태 탭으로 돌아간다(주소로 바로 열었으면 [전체]).
3. **When** 답을 적고 [답변 보내기], **Then** 처리 완료가 되고 "답변을 보냈어요. 문의한 회원에게 알림이 갔어요". 단추는 [답변 고치기]로 바뀐다.
4. **When** 처리 완료 문의에서 [답변 고치기], **Then** "답변을 고쳤어요". 처음 답한 시각은 그대로, 알림은 다시 가지 않는다.
5. 빈 답은 보내지 않는다("답변을 적어 주세요"). 그사이 문의가 없어졌으면 "없어진 문의예요".

## Requirements

- **FR-001** `POST /api/me/inquiries` 판정 순서: 401 `LOGIN_REQUIRED` → 400 `VALIDATION_FAILED`(형식) → 429 `TOO_MANY_REQUESTS`(하루 10건) → 400 `imageId`(내가 올리고 완료 확인까지 마친 글 사진이 아님, "사진을 다시 올려 주세요").
- **FR-002** 칸 오류는 모아서 한 번에 돌려준다. `category` 필수·네 종류 중 하나. `content`·`answer`는 앞뒤 공백을 뺀 뒤 1~2,000자(코드 포인트). `targetHandle`은 블로그 주소 형식(아니면 "블로그 주소를 확인해 주세요"). `imageId`는 없거나 1 이상 정수.
- **FR-003** 사진을 붙이면 `image`를 `ATTACHED`·`detached_at = NULL`로 바꿔 24시간 TEMP 정리에 걸리지 않게 한다. 사진이 지워지면 문의의 사진 칸은 비워진다(FK `SET NULL`).
- **FR-004** 관리자 API는 004 관리자 경로 규칙(비회원 401·일반 회원 404)이 먼저 막고, Service가 014 `ModerationAccess`로 다시 확인한다.
- **FR-005** 답 저장은 그 문의를 `FOR UPDATE`로 잠근다. 상태가 접수일 때만 `InquiryAnswered`를 발행하고, 커밋 뒤 비동기로 알림을 만든다(실패는 WARN만).
- **FR-006** 모든 응답은 `Cache-Control: private, no-store`.
- **FR-007** 탈퇴 정리 order 75에서 그 회원의 문의를 모두 지운다(멱등). 붙인 사진은 order 40 사진 정리가 연결 해제로 표시한다.

## 데이터 (V12__inquiry.sql)

- **§1 `inquiry`** `id` · `member_id`(FK member, CASCADE) · `category` varchar(20) · `content` text · `target_handle` varchar(39) NULL · `image_id`(FK image, SET NULL) · `status` 기본 `RECEIVED` · `answer` · `answered_by`(FK member, SET NULL) · `answered_at` · `created_at`.
- **§2 CHECK** 종류 4개, 상태 `RECEIVED`·`ANSWERED`, `ANSWERED`이면 `answer`·`answered_at`이 함께 찬다, `target_handle`은 `USER_REPORT`일 때만.
- **§3 인덱스** `ix_inquiry_member (member_id, id DESC)`(내 문의), `ix_inquiry_status (status, id DESC)`(관리자 목록).
- **§4 알림** `ck_notification_type`에 `INQUIRY_ANSWERED`를 더했다(지우고 다시 만듦, NOT VALID 뒤 VALIDATE). 끌 수 있는 종류는 그대로.
- **§5 Redis** `ratelimit:inquiry:{memberId}:1d` 하루 10건.

## API (contracts)

| 메서드 · 경로 | 본문 / 쿼리 | 성공 | 오류 |
|---|---|---|---|
| `POST /api/me/inquiries` (CSRF) | `{category, content, targetHandle?, imageId?}` | 201 문의 한 건 | 400 `VALIDATION_FAILED` · 401 · 429 `TOO_MANY_REQUESTS` |
| `GET /api/me/inquiries` | | 200 `{items}`(새것부터 50건) | 401 |
| `GET /api/admin/inquiries` | `status=RECEIVED\|ANSWERED`(없으면 전부), `before`(숫자 아니면 첫 쪽) | 200 `{items, nextBefore}`(20개씩) | 400 `status` `INVALID_VALUE` · 401 · 404 |
| `GET /api/admin/inquiries/{id}` | | 200 관리자 문의 한 건 | 401 · 404(없는 번호·숫자 아님·일반 회원) |
| `PUT /api/admin/inquiries/{id}/answer` (CSRF, POST도 같음) | `{answer}` | 200 관리자 문의 한 건 | 400 · 401 · 404(없는 번호·숫자 아님·일반 회원) |

문의 한 건: `{id, category, content, targetHandle, imageUrl, status, answer, answeredAt, createdAt}`. 관리자 쪽은 `member: {handle, nickname, withdrawn}`을 더한다(익명 처리면 주소·닉네임 `null`).

## 시험

- 백엔드: `InquiryApiIT`(보내면 접수·맨 위, 비회원 401, 칸 오류 목록, 내 사진만 붙음, 하루 한도 429, 일반 회원은 관리자 API 못 봄, 답하면 답변 완료·한 번만 알림·알림 주소 `/support?tab=mine`, 관리자 문의 하나 열기·없는 번호 404·일반 회원 404, 없는 문의 404), `InquiryInputTest`(공백 정리·칸 오류 모음·모르는 종류·2,000자·블로그 주소 정리·다른 종류면 버림·형식 오류·사진 번호·답변), `NotificationTypeTest`(운영 알림), `FlywayBaselineIntegrationTest`.
- 화면: `SupportPage.test.tsx`(비로그인 이동·목록·탭 [문의·신고하기]·[내 문의·신고 N]·빈 안내·보내면 맨 위·사용자 신고 칸·서버 칸 오류·한도·다시 불러오기), `InquiryForm.test.tsx`(사진 붙이기·빼기·실패 이유·2,000자), `AdminInquiriesPage.test.tsx`(메뉴 [문의]·한 줄 목록·누르면 문의 화면·처리 전/처리 완료 탭·[← 문의 목록]은 열었던 탭으로(주소로 바로 열면 전체)·답하면 처리 완료·빈 답·없는 문의·더 보기·다시 불러오기), `notificationText.test.ts`(답변 알림 문구).
