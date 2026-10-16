# Feature Specification: 설정 세로 탭 · 꼬리말 · 이용약관

**Feature Branch**: `027-settings-footer-terms`

**Created**: 2026-10-14

**Status**: Implemented (BaseLOG PR #118 · #120 — dev)

**Input**: 민서 요청(2026-10-14) 사이트 맨 아래에 GitHub 링크와 이용약관을 두고(PR #118), 설정 화면을 왼쪽 세로 탭으로 나누고 닉네임 조건과 사용 가능 여부를 바로 보여 달라(PR #120). 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-14

- Q: 설정 탭은 몇 개? → A: 왼쪽 세로 탭 다섯 개 — 프로필·계정·화면·알림·이용약관. 기본은 프로필. 각 탭 앞에 작은 그림(선 아이콘).
- Q: 주소는? → A: `/settings?tab=account|display|notifications|terms`. 프로필은 `?tab=` 없이 `/settings`. 모르는 값은 프로필로.
- Q: 탭에 무엇이 들어가나? → A: 프로필(사진·배너·닉네임·소개·둥지 주소), 계정(계정 정보·비밀번호·최근 활동 공개 + 사진 저장 공간), 화면(테마), 알림(알림 설정), 이용약관(AI 외부 전송 동의 + 약관 본문). 처음엔 [AI 연결] 탭이었다가 [이용약관]으로 바꿨다.
- Q: 다크 모드를 끈 빌드는? → A: 화면 탭이 빈 칸이라 뺀다. `?tab=display`도 프로필로 연다.
- Q: 탭을 옮기면 쓰던 내용은? → A: 남는다. 패널은 모두 그려 두고 고른 것만 보인다.
- Q: 닉네임 안내는? → A: 칸 아래에 조건을 늘 보이고, 입력이 0.5초 멈추면 서버에 사용 가능 여부를 묻는다. 지금 쓰는 닉네임과 같으면 묻지 않는다. 30일 제한 중이면 조건 대신 다음 변경 가능일을 보인다.
- Q: 꼬리말은? → A: 모든 화면 맨 아래 가운데에 "GitHub · 이용약관". GitHub는 우리 GitHub Organization을 새 탭으로, 이용약관은 `/terms`로. 글쓰기 화면(`/write/…`)에는 넣지 않는다.
- Q: 약관 본문은? → A: 확정 전 초안. `/terms` 화면과 설정 [이용약관] 탭이 같은 본문(`TermsContent`)을 쓴다. 두 곳 모두 "이 약관은 초안이에요. 확정되면 이 안내를 지워요".

### Session 2026-10-16 (폰 너비 탭)

- Q: 폰에서는 탭이 어디에 있나? → A: 폰 너비(640px 이하)에서는 한 줄 배치로 바뀌어 탭이 패널 위쪽에 가로 한 줄로 놓인다(PR #141). 탭은 나란히 늘어나고, 작은 그림은 숨기고, 위에 붙어 따라오지 않는다.
- Q: 폰에서 방향키는? → A: 탭 목록 `aria-orientation`이 `horizontal`이 되고 왼쪽·오른쪽 방향키로 앞뒤 탭을 옮긴다. 위·아래 방향키로는 옮기지 않는다. 넓은 화면은 그대로 `vertical`·위·아래(6defb46, `useHorizontalTabs`).

## User Scenarios & Testing

### User Story 1 - 설정 탭 옮기기 (P1)

1. **Given** `/settings`, **Then** 머리 "설정"과 "프로필과 계정, 화면, 알림, 이용약관을 한곳에서 관리해요.", 왼쪽에 탭 다섯 개. 프로필이 골라져 있다.
2. **When** [계정]을 누르면, **Then** 주소가 `/settings?tab=account`로 바뀌고 계정 패널만 보인다.
3. **When** 탭에서 위·아래 방향키, **Then** 앞뒤 탭으로 옮기며 고른다(끝에서 처음으로 돈다). Home·End는 처음·끝 탭.
3a. **Given** 폰 너비(640px 이하), **Then** 탭은 위쪽 가로 한 줄이고 `aria-orientation`은 `horizontal`. **When** 왼쪽·오른쪽 방향키, **Then** 앞뒤 탭으로 옮기며 고른다. 위·아래 방향키는 탭을 옮기지 않는다.
4. **Given** `/settings?tab=terms`, **Then** AI 동의와 그 아래 "이용약관" 본문(조 제목은 h3)이 보인다.
5. **Given** 비로그인, **Then** `/login?returnTo=%2Fsettings`로 보낸다.

### User Story 2 - 닉네임 조건·실시간 확인 (P1)

1. **Given** 프로필 탭, **Then** 닉네임 칸 아래 "한글·영문·숫자로 2~10자, 한글이나 영문을 1자 이상 넣어 주세요 (공백·특수문자 불가). 바꾸면 30일 동안 다시 바꿀 수 없어요".
2. **Given** 지금 닉네임 그대로, **Then** "지금 쓰는 닉네임이에요"(서버에 묻지 않음).
3. **When** 다른 닉네임을 쓰고 0.5초 멈추면, **Then** 쓸 수 있으면 "사용할 수 있어요", 아니면 이유(형식·글자 없음·예약어·금지어·중복).
4. **Given** 요청 제한(429)이나 네트워크 오류, **Then** 아무것도 보이지 않는다. 최종 판정은 [저장]이 한다.
5. **Given** 30일 제한 중, **Then** 칸을 막고 "닉네임은 바꾼 뒤 30일이 지나야 다시 바꿀 수 있어요. 다음 변경 가능일 YYYY.MM.DD".

### User Story 3 - 꼬리말과 이용약관 화면 (P2)

1. **Given** 아무 화면, **Then** 맨 아래 위쪽 선과 "GitHub · 이용약관". GitHub 앞에 GitHub 아이콘.
2. **When** [GitHub], **Then** `https://github.com/AIGJ006`을 새 탭으로 연다(`noopener noreferrer`).
3. **When** [이용약관], **Then** `/terms` — "이용약관", 버전·시행일, 초안 안내, 본문 8조(목적·계정·게시물과 권리·하지 말아야 할 일·신고와 이용 제한·탈퇴·서비스 변경과 책임·약관 변경).
4. **Given** 글쓰기 화면, **Then** 꼬리말이 없다.

## Requirements

- **FR-001** 설정 탭은 W3C 탭 패턴(`tablist` 세로, `tab`, `tabpanel`)을 따른다. 고른 탭만 `tabIndex` 0. 폰 너비(`(max-width: 640px)`, settings.css와 같은 조건)에서는 `useHorizontalTabs`로 `tablist`를 `horizontal`로 바꾸고 방향키도 왼쪽·오른쪽으로 바꾼다.
- **FR-002** 탭을 고르면 `?tab=`을 `replace`로 바꾼다(뒤로 가기 기록을 쌓지 않음).
- **FR-003** 넓은 화면 배치는 왼쪽 13rem 탭 + 오른쪽 패널. 탭 줄은 위에 붙어 따라온다(`sticky`). 폰 너비(640px 이하)는 한 줄 배치 — 탭은 패널 위에 가로 한 줄(`sticky` 아님, 탭마다 늘어나고 아이콘 숨김).
- **FR-004** 고른 탭·마우스를 올린 탭 바탕은 새 변수 `--color-nav-active-bg`(라이트 `#eef1f4`, 다크 `#2b2f33`), 고른 탭 글자는 `--color-brand`.
- **FR-005** 닉네임 확인은 입력이 0.5초 멈춘 뒤 한 번만 묻는다. 앞뒤 공백은 떼고 묻는다. 빈 값·지금 닉네임은 묻지 않는다. 안내가 없어도 한 줄 자리를 잡아 아래 칸이 밀리지 않는다.
- **FR-006** 서버의 닉네임 확인은 로그인한 회원이면 자기 닉네임을 중복에서 뺀다. 판정 순서: 형식 → 한글·영문 1자 → 예약어 → 금지어 → 중복(대소문자 무시).
- **FR-007** 꼬리말은 공통 틀(`AppLayout`)에 둔다. `/write/`로 시작하는 주소에서는 그리지 않는다.
- **FR-008** 약관 본문은 한 부품을 두 곳이 쓴다. 설정 탭 안에서는 조 제목을 h3로 낮춘다.
- **FR-009** `/terms`의 버전·시행일은 `GET /api/agreements/current`(비로그인 허용)에서 읽는다. 실패하면 "버전 정보를 불러오지 못했어요".
- **FR-010** 색은 tokens.css 변수만 쓴다.

## API (contracts)

새 API는 없다. 001에 있던 확인 API를 설정 화면도 쓴다.

`GET /api/nicknames/availability?nickname=...` (로그인 없이 가능, 로그인하면 자기 닉네임 제외)

```json
{"available": false, "code": "NICKNAME_DUPLICATE"}
```

- `code`: `NICKNAME_INVALID_FORMAT` · `NICKNAME_LETTER_REQUIRED` · `NICKNAME_RESERVED` · `NICKNAME_BANNED_WORD` · `NICKNAME_DUPLICATE`, 쓸 수 있으면 `null`.
- 요청 제한: IP당 1분 20회(`ip-limit-per-minute` 20, Redis 키 `rl:availability:nickname:ip:{ip}`). 넘으면 429.
- 64자보다 긴 값은 잘라서 판정한다(형식 오류가 된다).

## 시험

- 화면: `SettingsPage.test.tsx`(탭 다섯 개·기본 프로필·`?tab=`, 방향키·Home·End, 넓은 화면은 `vertical`, 폰 너비는 `horizontal`·왼쪽·오른쪽 방향키, 이용약관 탭에 AI 동의와 본문, 닉네임 조건과 "지금 쓰는 닉네임이에요", 30일 제한), `AvailabilityHint.test.tsx`(0.5초 뒤 한 번, 사용 가능, 429면 표시 없음, 빈 값, 지금 값이면 묻지 않음), `SiteFooter.test.tsx`(GitHub 새 탭, `/terms` 링크, 글쓰기 화면엔 없음).
- 백엔드: `AvailabilityIntegrationTest`, `NicknamePolicyTest`(닉네임 판정).
- e2e: `account-responsive.spec.ts`(설정 화면 가로 스크롤 없음), `theme.spec.ts`·`notification.spec.ts`·`withdraw.spec.ts`(해당 탭 주소 `?tab=…`로 바로 열도록 고침).
