# Feature Specification: 맞춤법·띄어쓰기 검사

**Feature Branch**: `022-spell-check`

**Created**: 2026-10-14

**Status**: Implemented (BaseLOG PR — dev)

**Input**: 민서 요청(2026-10-14) "맞춤법이랑 띄어쓰기 검사 기능 추가 가능해?", "그 한글프로그램처럼 그냥 맞춤법이 틀린부분에 빨간줄이 그어지게 한다던가 하는 방법은?", 추천안에 "그렇게 하자 그럼". 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-14 (추천안대로 결정)

- Q: 어떻게 검사하나? → A: 두 겹. ① 제목·본문 칸에 브라우저 기본 맞춤법 검사(`spellcheck`, `lang="ko"`)를 켠다 — 브라우저·OS가 한국어 사전을 갖고 있으면 쓰는 동안 바로 빨간 밑줄이 생긴다(무료, 서버 안 부름). ② 글쓰기 머리 줄에 **[맞춤법 검사]** 버튼. 누르면 AI가 본문을 한 번 검사하고 틀린 곳에 빨간 물결 밑줄을 긋는다.
- Q: 쓰는 동안 자동으로 AI 검사? → A: 하지 않는다. 무료 한도(Gemini) 때문에 **버튼을 누를 때만** 부른다.
- Q: 고치는 방법은? → A: 밑줄 친 곳을 누르면(커서를 두면) 그 아래에 "틀린 말 → 고친 말 · 이유" 창과 [바꾸기]·[무시]. 결과 줄에 [모두 바꾸기]·[검사 끝내기]. **글을 저절로 바꾸지 않는다.** [바꾸기]는 Ctrl+Z로 되돌릴 수 있다.
- Q: 어떤 AI? → A: 013 AI 태그 추천과 같다. 배포 서버는 Gemini(`gemini-flash-lite-latest`), 자체 AI(Ollama)는 켜 둔 곳(로컬)에서만. 같은 키라 Gemini 하루 한도(450)와 쉬는 시간 상태를 함께 쓴다.
- Q: 동의는? → A: 013과 같은 AI 동의를 쓴다. 문구에 맞춤법 검사를 더하고 버전을 2026-10-14로 올렸다(모든 회원이 다음에 쓸 때 한 번 다시 동의). 비공개·친구 공개 글은 외부로 보내지 않는다 — 자체 AI가 꺼진 배포 서버에서는 "비공개·친구 공개 글은 지금 검사할 수 없어요".
- Q: 한도·오류 때는? → A: 회원당 하루 30회(태그 추천과 따로, 한국 시간 0시 초기화). 한도 초과·AI 오류는 결과 줄에 안내 문구만 뜨고 글쓰기·자동 저장·발행에는 영향이 없다. AI가 답을 주지 못한 요청은 횟수에서 빼 준다. 횟수는 공급자를 부르기 전에 Redis Lua 한 번으로 "올리고 한도 넘으면 되돌리기"를 원자적으로 해서, 동시에 여러 번 눌러도 30회를 넘지 않는다.
- Q: 제목도? → A: AI 검사는 본문만(밑줄을 본문 칸에 긋기 때문). 제목은 브라우저 기본 검사만.

## User Scenarios & Testing

### User Story 1 - 버튼으로 검사하고 밑줄 보기 (P1)

1. **Given** 본문에 "할수있다고", "안되요", **When** [맞춤법 검사], **Then** 두 곳에 빨간 물결 밑줄, 결과 줄 "고칠 곳 2개 · 빨간 밑줄을 누르면 고친 말이 보여요", "AI 제안이라 틀릴 수 있어요 · 오늘 남은 검사 29회". 본문은 그대로.
2. **Given** 같은 틀린 말이 여러 번, **Then** 모두 밑줄. 코드 블록·인라인 코드·링크 주소 안은 밑줄을 긋지 않는다(서버도 보내지 않는다).
3. **Given** 본문이 5,000자(Gemini) / 1,500자(자체 AI)를 넘으면, **Then** 앞부분만 검사하고 "본문 앞부분만 검사했어요".
4. **Given** 틀린 곳이 없으면, **Then** "고칠 곳을 찾지 못했어요".

### User Story 2 - 고치기 (P1)

1. **When** 밑줄 친 곳을 누르면, **Then** "안되요 → 안 돼요 · '돼요'가 맞아요" 창과 [바꾸기]·[무시].
2. **When** [바꾸기], **Then** 그 자리만 바뀌고 밑줄이 사라진다. 자동 저장이 평소처럼 돈다.
3. **When** [무시], **Then** 그 밑줄만 사라진다. 직접 고치거나 지운 곳의 밑줄도 사라지고, 앞에 글을 더 써도 나머지 밑줄은 제자리를 따라간다.
4. **When** [모두 바꾸기], **Then** 남은 곳을 한 번에 바꾼다(되돌리기 한 번으로 돌아감). 다 고치면 "다 고쳤어요".

### User Story 3 - 안 될 때 (P2)

1. 동의가 없으면 동의 창 → [동의하고 검사하기] 뒤 같은 본문으로 다시 검사. [취소]는 아무것도 보내지 않는다.
2. 오늘 30회를 다 쓰면 "오늘 맞춤법 검사를 모두 썼어요. 내일 다시 써 보세요".
3. AI 시간 초과·오류·혼잡·Redis 장애면 "지금은 검사할 수 없어요. 잠시 뒤 다시 시도해 주세요".
4. 검사할 글자가 없으면(코드만 있는 글 등) "검사할 내용이 없어요".

## Requirements

- **FR-001** 제목·본문 칸은 `spellcheck`와 `lang="ko"`를 켠다.
- **FR-002** AI 검사는 [맞춤법 검사]를 누를 때만 한다. 쓰는 동안 자동으로 부르지 않는다.
- **FR-003** `POST /api/posts/{postId}/spell-check`는 내 글(휴지통 밖)에만. 판정 순서 401 → 403 계정 상태 → 404 → 503 DISABLED → 409 동의 → 400 본문 길이(글 최대 길이 `blog.post.content-max` 100,000자를 넘으면 `CONTENT_TOO_LONG`) → 422 짧음 → 503 NOT_PUBLIC → 429 하루 한도 → 공급자.
- **FR-004** 공개 범위가 PUBLIC이 아니면 외부 AI로 보내지 않는다(013 FR-030과 같음).
- **FR-005** 서버는 AI가 준 고칠 곳 중 보낸 글에 그대로 있는 것만, 같은 틀린 말은 하나만, 최대 50개 돌려준다.
- **FR-006** 서버는 글을 바꾸거나 저장하지 않는다(DB 쓰기 없음). 화면도 사용자가 누를 때만 바꾼다.
- **FR-007** 로그는 요청마다 한 줄(글 번호·공급자·결과·개수·길이·걸린 시간). 본문과 고칠 말은 남기지 않는다.
- **FR-008** 화면 색은 tokens.css 변수만(밑줄 `--color-danger`, 바탕 `--color-danger-bg`).

## 설계 결정 (research)

- **R1 서비스** — 새 모듈 `spellcheck`. 013 `tag.application.suggest`의 공개 클래스(ProviderState·AiRedis·설정)만 쓴다(헌법 II).
- **R2 공급자** — 013 라우터와 같은 규칙: 비공개면 자체 AI만, Gemini 키 없음·소진·쉬는 중이면 자체 AI, Gemini 429면 자체 AI로 다시, 시간 초과·서버 오류면 60초 쉬게 하고 실패. 응답 시간 제한은 검사용으로 길게(Gemini 30초, 자체 AI 60초).
- **R3 입력 정리** — 글자는 바꾸지 않고 코드 블록·인라인 코드·사진·HTML 태그·맨 주소를 덜어 내고, 링크는 글자만. 줄은 그대로. 공급자 한도(Gemini 5,000자, 자체 AI 1,500자)보다 길면 오류가 아니라 앞부분만 검사한다: 한도 안 마지막 줄바꿈이 한도의 절반보다 뒤에 있으면 거기서, 아니면(줄바꿈이 없거나 너무 앞) 한도 글자 수에서 자른다(글자 단위, 한글이 반으로 갈리지 않음).
- **R4 결과 거르기** — 틀린 말이 비었거나 보낸 글에 없으면(모델이 지어냄) 버림. 고친 말이 비었거나 같으면 버림. 틀린 말 60자·고친 말 80자 넘으면 버림. 이유는 100자까지.
- **R5 위치** — AI가 주는 위치는 믿지 않는다. 화면이 본문에서 틀린 말을 모두 찾는다(코드·링크 주소 안 제외, 겹치면 앞의 것만). 본문이 바뀌면 앞뒤 같은 부분을 빼고 바뀐 곳 하나를 찾아, 뒤의 밑줄은 옮기고 걸친 밑줄은 버린다.
- **R6 밑줄 그리기** — textarea 안에는 밑줄을 그을 수 없어서, 본문 칸 뒤에 같은 글을 같은 글꼴·여백·크기로 겹쳐 놓고(스크롤도 따라감) 그 층에 밑줄을 그린다. 본문 칸 배경만 투명하게 한다. 쓰기·커서·되돌리기는 본문 칸 그대로.

## 데이터 (data-model)

- **§1 고칠 곳** `{original, suggestion, reason}` — 저장하지 않는다.
- **§2 Redis** `ai:spell:usage:{memberId}:{yyyyMMdd 한국 시간}` 하루 사용 수(TTL 2일). Gemini 상태 키(`ai:gemini:*`)·자체 AI 동시 처리 키(`ai:ollama:inflight`)는 013과 함께 쓴다.
- **§3 503 이유** `details.reason` = `DISABLED` · `FAILED` · `BUSY` · `NOT_PUBLIC` · `STORE_UNAVAILABLE`.
- **§4 설정** `blog.ai.spell-check.*`: `enabled` true, `min-input-chars` 5, `max-issues` 50, `daily-limit-per-member` 30, `gemini.max-input-chars` 5000 · `timeout` 30s · `max-output-tokens` 4096, `ollama.max-input-chars` 1500 · `timeout` 60s · `max-output-tokens` 1024. 주소·모델·키는 `blog.ai.tag-suggest.gemini/ollama`.

## API (contracts)

`POST /api/posts/{postId}/spell-check` (CSRF, 로그인) 본문 `{"contentMd": "..."}`

200 (`Cache-Control: no-store`)

```json
{"issues":[{"original":"안되요","suggestion":"안 돼요","reason":"'돼요'가 맞아요"}],
 "provider":"GEMINI","truncated":false,"remainingToday":29}
```

| 상태 | code | details |
|---|---|---|
| 400 | `VALIDATION_FAILED` (`errors[0].code` `CONTENT_TOO_LONG`) | |
| 404 | 남의 글·없는 글·숫자 아님 | |
| 409 | `AI_CONSENT_REQUIRED` | `{version}` |
| 422 | `CONTENT_TOO_SHORT` | `{minChars, length}` |
| 429 | `AI_DAILY_LIMIT` + `Retry-After` | `{resetAt}` |
| 503 | `AI_UNAVAILABLE` | `{reason}` |

오류 본문은 공통 규칙대로 `errors`는 늘 목록(비면 `[]`), 문구 끝 마침표 없음.

## 공급자 (providers)

- **§1 프롬프트** — 고정 지시문(한국어 맞춤법·띄어쓰기·표준어만, original은 글 그대로 어절 단위, 영어·코드·고유명사·마크다운 기호·문체는 고치지 않음, 확실하지 않으면 넣지 않음, 최대 N개, JSON만) + "검사할 글:\n" + 정리된 본문.
- **§2 출력 스키마** — `{"issues": [{"original": string, "suggestion": string, "reason": string}], maxItems N}`. 형식이 다르면 MALFORMED(실패, 횟수 되돌림).
- **§3 Gemini** — `generateContent`, 키는 `x-goog-api-key` 헤더, `responseSchema`, `temperature` 0. 429는 `QuotaFailure.violations[].quotaId`로 하루·분당을 가른다.
- **§4 Ollama** — `/api/chat`, `format`에 스키마, `stream: false`, `num_predict` 1024.

## 시험

- 백엔드: `SpellCheckApiIT`(Gemini·비공개→자체 AI·NOT_PUBLIC·429 전환·실패 되돌림·409·422·404·401·하루 한도·자르기), `SpellInputCleanerTest`, `SpellIssueFilterTest`, `GeminiSpellCheckerTest`, `OllamaSpellCheckerTest`. 가짜 공급자는 `support/ai/FakeSpellCheckConfiguration`.
- 화면: `spellRanges.test.ts`(찾기·옮기기·모두 바꾸기), `SpellCheck.test.tsx`(누를 때만 부름·밑줄·바꾸기·무시·직접 고침·모두 바꾸기·앞부분·오류 문구·동의 흐름).
