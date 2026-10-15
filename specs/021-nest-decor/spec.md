# Feature Specification: 둥지 꾸미기·새 키우기

**Feature Branch**: `021-nest-decor`

**Created**: 2026-10-13

**Status**: Implemented (BaseLOG PR #94)

**Input**: 민서 요청(2026-10-13) "우리가 둥지니까 새를 키우거나", "연속 출석하면 새 크기가 커지고 출석 안 하면 줄어들고, 좋아요를 많이 받거나 해서 조건을 만족하면 포인트를 주고 그 포인트로 둥지나 새를 꾸미는 거야", "해줘". 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-13 (추천안대로 결정, 민서 위임)

- Q: 바탕은? → A: "둥지 꾸미기". 포인트를 쌓아 상점에서 사고 장착한다. 처음엔 코드(SVG)로 그렸으나 2026-10-13 민서 님이 픽셀 그림(새 4단계·둥지 9·새 장식 12·소품 9)을 보내 그것으로 바꿨다. 처음 모습은 기본 숲·기본 둥지. 성장 상태 UI(Lv·행복도 막대)는 넣지 않는다(민서 "무시해도 돼"). 2026-10-14 새 4단계와 둥지를 민서 님이 새로 준 큰 그림으로 바꾸고 단풍 둥지(200점)를 더했다.
- Q: 새 장식 위치가 단계마다 안 맞으면? → A: 2026-10-13 민서 님 결정으로 안경·선글라스·모자·목도리·방울 목걸이는 뺐다. 뒤이어 새 둘레의 나비·하트·별도 뺐다. 남은 새 장식은 4개(머리 칸 꽃관·나비 리본·왕관, 꽃핀). 2026-10-14 민서 님이 착용 그림(옆모습, 새 4단계)까지 그려 줘서 얼굴 칸 선글라스 2종을 더했다 — 단계마다 착용 그림을 따로 잘라 씌운다(`art/<코드>-<단계>.png`), 상점 그림은 앞모습. 2026-10-15 밀짚모자(머리 칸)를 더했다 — 민서 님 그림 속 "모자 쓴 새"를 단계마다 통째로 잘라 쓰면 새 그림을 그것으로 바꿔 그린다(4단계는 우리 새에 맞춰 다시 그린 그림). 선글라스는 그 새의 눈 위치로 옮긴다.
- Q: 출석 기준은? → A: 로그인 유지가 아니라 하루(Asia/Seoul) 한 번 **내 둥지(/@나)** 나 **[내 둥지 꾸미기]** 화면에 들어오면 출석. 처음이면 "출석했어요 +10" 알림.
- Q: 새 크기는? → A: 1~4단계(아기 새·꼬마 새·어린 새·어른 새 — 그림 4장에 맞춤). 연속 출석이 3의 배수가 되는 날 한 단계 자란다. 마지막 출석 다음 날부터 빠진 날이 3일을 넘으면 넘은 날 하루마다 한 단계씩 작아지고 1단계 밑으로는 안 내려간다. 작아진 크기는 읽을 때 셈한다.
- Q: 좋아요 포인트는? → A: 다른 사람이 누른 좋아요만, 같은 (글, 누른 사람)은 한 번만(취소 후 다시 눌러도 그대로), 하루 상한.
- Q: 포인트 저장은? → A: 잔액 칸 없이 "언제·왜·몇 점" 기록 표의 합. (회원, source_key) 유일로 중복 지급을 막고, 사기는 음수 한 줄, 되돌리기는 반대 부호 한 줄(ADJUST).
- Q: 어디에 보이나? → A: 계정 메뉴 [내 둥지 꾸미기] `/nest`(나만). 홈 화면(민서 "홈화면은 빼자")과 글 상세 작성자 카드(민서 "없어도 될 거 같은데")에는 넣지 않는다. 둥지 화면 머리말 오른쪽 새 칸도 뺐다(2026-10-14 민서 "휑해서 없느니만 못하다" → "걍빼자"). 내 둥지에 들어올 때 출석 기록은 그대로.
- Q: 배지(첫 글·7일 연속 등)? → A: 첫판에서 뺀다.

### Session 2026-10-14 (단추 글자·포인트 기록, PR #126)

- Q: 꾸미기 단추 글자는? → A: 새 장식은 [착용]·[착용 해제](착용 해제는 채운 초록), 배경·둥지는 [적용]·"적용 중"(누를 수 없음). 아직 없는 물건은 "🔒 N점으로 해금"(채운 초록)이고 누르면 사고 바로 씌운다. 화면 이름과 계정 메뉴 글자는 "새 키우기"(`/nest`).
- Q: 포인트 기록은 몇 개 보이나? → A: 처음 5개, [더 보기]마다 10개씩 더. API는 그대로 30개씩 주고, 받은 것을 다 보이면 다음 쪽(`before`)을 부른다.

## 정한 숫자 (`blog.nest.*`)

| 항목 | 값 |
|---|---|
| 출석 | +10 (하루 한 번) |
| 연속 출석 보너스 | 7일마다 +30 |
| 글 등록(첫 발행) | +20, 하루 3건 |
| 댓글·답글 | +5, 하루 5건 |
| 받은 좋아요 | +2, 하루 20건 |
| 성장 주기 | 연속 3일마다 한 단계 |
| 작아짐 | 3일 넘게 빠지면 하루 한 단계 |

| 상점 | 값 |
|---|---|
| 배경 | 기본 숲 0 · 풀밭 100 · 맑은 하늘 100 · 노을 150 · 벚꽃 200 · 밤하늘 250 |
| 둥지(하나) | 기본 0 · 나뭇잎 100 · 꽃 120 · 눈 150 · 리본 180 · 단풍 200 · 별 300 |
| 새 장식 — 머리(하나) | 꽃관 100 · 나비 리본 80 · 왕관 300 · 밀짚모자 150 |
| 새 장식 — 얼굴(하나) | 하트 선글라스 150 · 픽셀 선글라스 150 |
| 새 장식 — 자리 없음 | 꽃핀 70 |

값이 0인 물건은 사지 않아도 가진 것으로 본다. 새집·울타리·버섯 둥지, 소품 9종, 새 장식 하트·별·나비는 빠졌다(2026-10-13 민서). 뺀 물건을 샀던 사람에게는 쓴 포인트를 ADJUST 한 줄씩으로 돌려준다(V11, source_key `REFUND:<코드>`). 관리자(ADMIN) 계정에는 그때 팔던 물건 값의 합 2890점을 ADJUST 한 줄로 한 번 넣었다(V10, source_key `admin-grant:2026-10-13`).

## User Scenarios & Testing

### User Story 1 - 출석하고 새 키우기 (P1)

1. **Given** 오늘 처음 내 둥지에 들어오면, **Then** +10점과 "출석했어요 +10" 알림. 같은 날 다시 들어오면 포인트·알림 없음.
2. **Given** 어제까지 연속 2일, **When** 오늘 출석하면, **Then** 연속 3일·새가 한 단계 자라고 알림에 "새가 꼬마 새로 자랐어요".
3. **Given** 연속 6일, **When** 7일째 출석하면, **Then** +10 +30.
4. **Given** 4단계 새가 6일 전 마지막 출석, **Then** 지금 2단계로 보이고 연속은 0. 오늘 출석하면 연속 1, 2단계 그대로.

### User Story 2 - 포인트로 꾸미기 (P1)

1. **Given** 잔액이 값보다 적으면, **Then** 단추가 "🔒 N점 모자라요"로 막히고 API는 409 `NEST_POINTS_NOT_ENOUGH`.
2. **When** "🔒 N점으로 해금"을 누르면, **Then** 확인 창 없이 사고 바로 씌운다. 잔액이 줄고 기록에 "물건 사기 · 노을 −150". 같은 물건을 다시 사면 409 `NEST_ITEM_ALREADY_OWNED`. (2026-10-14 고침: 전에는 [사기] → 확인 창 → [장착])
3. **When** 배경·둥지에서 [적용]을 누르면, **Then** 하나로 바뀌고 그 물건 단추는 "적용 중"(누를 수 없음). 새 장식에서 [착용]을 누르면 더해지고(같은 자리 새 장식은 화면이 바꿔 단다, 서버는 두 개 오면 400 `NEST_SLOT_TAKEN`) 단추는 초록 [착용 해제]가 된다. 사지 않은 물건은 400 `NEST_ITEM_NOT_OWNED`. (2026-10-14 고침: 전에는 [장착]·[내리기])
4. **Given** 포인트 기록이 많으면, **Then** 처음 5개만 보이고 [더 보기]마다 10개씩 더 보인다. (2026-10-14)

### User Story 3 - 둥지와 새 보여 주기 (P2)

1. ~~남의 둥지 화면 오른쪽에 그 주인의 둥지·새·크기 이름·연속 출석.~~ 2026-10-14 뺌: 새는 [새 키우기] 화면에서만 보인다. `GET /api/members/{handle}/nest`는 서버에 남아 있지만 화면은 부르지 않는다.

### 활동 포인트 (FR-020~FR-025)

- 글 첫 발행(`PostPublished`)·댓글(`CommentCreated`)·좋아요(`PostLiked`) 이벤트를 커밋 뒤 비동기로 받아 준다. 이벤트가 빠져도 핵심 기능은 그대로(constitution V).
- 하루 상한은 서비스 시간대 날짜(`earned_on`)로 센다. 같은 회원의 지급·출석·사기는 둥지 줄 잠금으로 한 줄로 세운다.

## Data Model (Flyway V9, 새 표만)

- `nest` (회원당 1줄): `member_id` PK·FK, `bird_level` 1~4, `streak_days`, `best_streak_days`, `last_attended_on` date, `background`, `nest_type`, `created_at`, `updated_at`.
- `nest_point`: `id`, `member_id`, `reason`(ATTENDANCE·STREAK_BONUS·POST·COMMENT·LIKE_RECEIVED·PURCHASE·ADJUST), `amount`(≠0), `source_key`, `earned_on`, `created_at`. UNIQUE(`member_id`, `source_key`). source_key 예: `ATTEND:2026-10-13`, `STREAK:2026-10-13`, `POST:{postId}`, `COMMENT:{commentId}`, `LIKE:{postId}:{likerId}`, `BUY:{itemCode}`.
- `nest_item`: PK(`member_id`, `item_code`), `equipped`(새 장식만), `bought_at`.
- 탈퇴 정리 order 25가 세 표에서 그 회원 줄을 지운다.

## API

| 이름 | 요청 | 응답 |
|---|---|---|
| getMemberNest | `GET /api/members/{handle}/nest` (누구나) | `{level, levelName, streakDays, background, nestType, decorations[]}` · 없는 주인 404 |
| getMyNest | `GET /api/me/nest` | `{nest, balance, attendedToday, bestStreakDays, daysToGrow, today[{reason,count,cap,points}], items[{code,kind,name,price,owned,equipped}]}` |
| attend | `POST /api/me/nest/attendance` | `{attended, pointsGained, grew, me}` · 같은 날 다시 부르면 `attended:false` |
| listMyNestPoints | `GET /api/me/nest/points?before={id}` | `{items[{id,reason,label,amount,createdAt}], nextBefore}` 30개씩 새것부터 (화면은 5개 → [더 보기]마다 10개씩 보이고, 받은 것을 다 보이면 다음 쪽을 부른다) |
| buyNestItem | `POST /api/me/nest/items/{code}` | MyNest · 400 `NEST_ITEM_UNKNOWN`, 409 `NEST_ITEM_ALREADY_OWNED`·`NEST_POINTS_NOT_ENOUGH` |
| equipNest | `PUT /api/me/nest/equipped` `{background, nestType, decorations[]}` | MyNest · 400 `NEST_ITEM_NOT_OWNED`·`NEST_SLOT_TAKEN` |

시험용 `PUT /api/me/nest/dev/level`·`POST /api/me/nest/dev/points`는 `blog.nest.dev-tools=true`(로컬 설정)일 때만 생긴다.

`/api/me/**`는 로그인 필요(401), 쓰기는 정지·탈퇴 유예 403(`ACCOUNT_WRITE`). 응답 `Cache-Control: private, no-store`(남의 둥지는 `no-cache`).

## 화면 규칙

- 그림은 `frontend/src/features/nest/art/*.png`(물건 코드 이름). 둥지 그림은 뒤쪽 통째·앞쪽 아래만 두 번 놓아 새가 둥지 안에 앉아 보이게 한다. 배경색은 `tokens.css`의 `--nest-bg-*`·`--nest-ground` 라이트·다크 쌍만. 화면 문구는 "블로그" 대신 "둥지".
- 화면 이름은 "새 키우기"(`/nest`, 계정 메뉴 같은 글자). 꾸미기는 [배경]·[둥지]·[새 장식] 탭.
- 단추(2026-10-14, PR #126): 없는 물건 "🔒 N점으로 해금"(모자라면 "🔒 N점 모자라요"로 막힘), 배경·둥지 [적용] / "적용 중", 새 장식 [착용] / [착용 해제]. 해금 단추와 [착용 해제]는 채운 초록(`--color-brand-fill`).
- 포인트 기록: 처음 5개, [더 보기]마다 10개씩. 빨리 두 번 눌러도 같은 기록이 겹치지 않게 번호로 거른다.
