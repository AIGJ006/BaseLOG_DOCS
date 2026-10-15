# Feature Specification: 관리자 회원 목록

**Feature Branch**: `026-admin-members`

**Created**: 2026-10-13

**Status**: Implemented (BaseLOG PR #86, #89 — dev)

**Input**: 민서 요청(2026-10-13) 관리자 화면에서 가입 회원 목록을 보고 찾기, 머리말에 관리자 화면으로 가는 [관리]. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-13

- Q: 어디에 보이나? → A: `/admin/members`. 관리자 화면 위쪽 메뉴에 **[회원 목록]**. 관리자만 본다(004 `AdminRouteGate`, 서버 `AdminPathSecurityCustomizer`).
- Q: 어떤 순서로? → A: 가입 최근 순, 한 쪽 20명. [이전]·[다음]으로 넘긴다.
- Q: 누가 빠지나? → A: 탈퇴 신청(유예) 회원과 익명 처리된 회원. 목록·전체 수·상태 거르기 모두에서 뺀다. 관리자는 목록에 나온다("관리자" 표시).
- Q: 무엇으로 찾나? → A: 주소·닉네임·이메일 부분 일치(대소문자 무시). `%`·`_`·역슬래시는 글자 그대로 찾는다. 상태는 전체·정상·정지 중에서 거른다.
- Q: 줄마다 무엇을? → A: 닉네임(회원 한 명 화면으로 이동)·@주소·관리자/정지 표시·이메일(없으면 "이메일 없음")·로그인 방식·가입일·마지막 로그인(없으면 "로그인 기록 없음"). 이메일은 이 화면에만 나온다.
- Q: 정지는? → A: 목록에서 하지 않는다. 이름을 누르면 014 회원 한 명 화면(`/admin/members/:handle`)에서 정지·해제·이력을 본다. 그 화면에도 같은 메뉴가 붙는다.
- Q: 관리자 화면으로는 어떻게 가나? → A: 머리말 로고 옆 [공지]·[업데이트 소식] 다음에 관리자에게만 **[관리]**(`/admin`). `/admin`은 `/admin/reports`로 넘긴다.

## 관리자 화면 구성 (지금 dev)

- 위쪽 메뉴(`AdminNav`, "관리자 메뉴"): **[신고 관리]** `/admin/reports` · **[문의]** `/admin/inquiries` · **[회원 목록]** `/admin/members` · **[통계]** `/admin/stats`. 지금 화면의 메뉴가 켜진다. 회원 한 명 화면에서도 [회원 목록]이 켜진다.
- 메뉴가 붙는 화면: 신고 관리, 문의, 회원 목록, 회원 한 명, 통계. 신고 한 건 화면(`/admin/reports/:caseId`)에는 없다.
- 문의는 PR #121, 통계는 024(PR #129)에서 더했다.

## User Scenarios & Testing

### User Story 1 - 목록 보기 (P1)

1. **When** 관리자가 [회원 목록], **Then** "전체 N명"과 가입 최근 순 20명.
2. **Given** 탈퇴 신청·익명 처리 회원, **Then** 목록과 전체 수에 없다.
3. **Given** 소셜 로그인이 이메일을 주지 않은 회원, **Then** "이메일 없음"과 "GitHub 로그인".
4. **When** 이름을 누르면, **Then** `/admin/members/{handle}`(정지·이력).

### User Story 2 - 찾기·거르기 (P1)

1. **When** "KIM"으로 찾기, **Then** 주소 `kim001` 회원. 닉네임·이메일 일부로도 찾는다.
2. **When** 상태 [정지]를 고르면, **Then** 바로 찾고 첫 쪽부터. 검색어·상태·쪽은 주소창(`?q&status&page`)에 담긴다.
3. **When** `a_b`로 찾기, **Then** `a_b001`만(밑줄을 글자로). `%`는 아무도 없음.
4. 찾는 회원이 없으면 "찾는 회원이 없어요", 회원이 없으면 "회원이 없어요". 불러오지 못하면 "불러오지 못했어요" + [다시 불러오기].

### User Story 3 - 권한·머리말 (P1)

1. 비로그인 → 401(화면은 로그인으로), 일반 회원 → 404.
2. 머리말 [관리]는 관리자에게만 보인다.

## Requirements

- **FR-001** `GET /api/admin/members?q&status&page`는 관리자만(014 `ModerationAccess.requireAdmin`). 응답은 이메일이 담기므로 늘 `no-store`.
- **FR-002** 쪽 크기 20. 쪽은 1부터, 마지막 쪽을 넘으면 빈 목록(`page`는 그대로).
- **FR-003** 400: 검색어 100자 초과(`q` `TOO_LONG`), 상태가 `ACTIVE`·`SUSPENDED` 밖(`WITHDRAWN` 포함, `status` `INVALID_VALUE`), 쪽이 1 미만·100,000 초과·숫자 아님(`page` `INVALID_VALUE`).
- **FR-004** 검색어는 앞뒤 공백을 지운다. 공백뿐이면 찾지 않고 전부.
- **FR-005** 응답에 회원 번호(`id`)와 비밀번호는 없다.
- **FR-006** 화면 검색 칸은 100자까지(`maxLength`).

## 데이터

- 새 테이블 없음. `member`와 `auth_identity`를 읽는다(account `MemberQueryService.searchAdminList`·`countAdminList`).
- 조건: `deleted_at IS NULL AND status <> 'WITHDRAWN'`, 상태, `handle`·`nickname`·`email` `ILIKE` 부분 일치. 정렬 `created_at DESC, id DESC`.

## API

`GET /api/admin/members?q=kim&status=SUSPENDED&page=1` (로그인, 관리자) → 200 (`Cache-Control: private, no-store`)

```json
{"items":[{"handle":"kim001","nickname":"김민서","email":"kim@example.com","provider":"LOCAL",
  "role":"USER","status":"ACTIVE","joinedAt":"...","lastLoginAt":"2026-10-01T00:00:00Z"}],
 "page":1,"size":20,"total":3,"totalPages":1}
```

`provider`는 `LOCAL`·`GITHUB`·`GOOGLE`(없으면 null), `email`·`lastLoginAt`은 없으면 null.

| 상태 | 경우 |
|---|---|
| 400 | `VALIDATION_FAILED` (`q`·`status`·`page`) |
| 401 | 비로그인 |
| 404 | 관리자 아님 |

회원 한 명·정지·해제(`GET /api/admin/members/{handle}`, `POST …/suspensions`, `DELETE …/suspensions/current`)는 014 그대로다.

## 시험

- 백엔드: `AdminMemberListIT`(가입 최근 순·이메일·로그인 방식·탈퇴 신청·익명 처리 제외, 주소·닉네임·이메일 찾기와 상태 거르기, `%`·`_` 글자 그대로, 20명씩·마지막 쪽 넘으면 빈 목록, 형식 오류 400), `AdminAccessIT`(`/api/admin/members`·`/admin/members` 비회원 401·일반 회원 404·관리자 통과).
- 화면: `AdminMembersPage.test.tsx`(전체 수·이메일·가입 방식·상태·회원 화면 링크, 검색어·상태가 주소창과 요청에 담김, [이전]·[다음], 없음 안내·[다시 불러오기]), `SiteHeader.test.tsx`([관리]는 관리자에게만).
