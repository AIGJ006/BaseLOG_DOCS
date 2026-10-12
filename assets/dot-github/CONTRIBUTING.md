# BaseLOG 작업 방법

BaseLOG(https://baselog.java21.net)는 강성찬·나민서·김민서가 만드는 티스토리 같은 블로그 서비스입니다.
이 문서는 팀 저장소 전체에 적용되는 작업 방법입니다.

## 저장소

| 저장소 | 내용 |
|---|---|
| [BaseLOG](https://github.com/AIGJ006/BaseLOG) | 구현 코드: `backend/`(Spring Boot + PostgreSQL + Flyway), `frontend/`(React), `deploy/`, 배포 workflow |
| [BaseLOG_DOCS](https://github.com/AIGJ006/BaseLOG_DOCS) | 설계 문서: 원문 `docs/01~51`, 기능별 `specs/NNN-기능/`, 원칙 `.specify/memory/constitution.md`, Spec Kit 명령 |

코드 주석의 `docs/NN`, `specs/NNN-기능` 경로는 BaseLOG_DOCS 기준입니다.

## 새 기능은 스펙부터

[GitHub Spec Kit](https://github.com/github/spec-kit)으로 스펙을 먼저 쓰고 구현합니다.

1. **BaseLOG_DOCS**에서 `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks`로 `specs/NNN-기능/`을 만듭니다.
2. 팀과 정한 것은 BaseLOG_DOCS README의 **"정해진 것"** 표에 날짜와 함께 적습니다. 공통 스펙(001~016)을 바꿀 때는 팀에 먼저 묻습니다.
3. **BaseLOG**에서 `/speckit-implement`로 tasks를 따라 구현합니다.
4. API 계약(`openapi.yaml`)을 바꾸면 BaseLOG의 `backend/src/test/resources/contracts/<기능>/openapi.yaml` 복사본도 같이 바꿉니다. 계약 시험이 이 복사본을 읽습니다.

## 브랜치와 배포 (BaseLOG)

- **`dev`**: 작업 브랜치. 기능·수정마다 브랜치를 만들고 **PR을 `dev`로** 올립니다. dev에 머지해도 배포되지 않습니다.
- **`main`**: 배포 브랜치. 릴리스 때 **`dev` → `main` PR** 하나를 머지하면 서버에 자동 배포됩니다.
  이 PR에 릴리스 노트 `frontend/src/features/site-news/releases/vX.Y.Z.md`를 함께 넣으면 GitHub 릴리스와 사이트 "업데이트 소식"이 생깁니다.
- **급한 고침**(사이트가 안 열리는 등)만 `main`으로 바로 PR을 올리고, 머지 뒤 `main`을 `dev`에 합칩니다.
- **되돌리기**: `main`에서 릴리스 머지 커밋을 revert하는 PR을 머지하면 직전 버전으로 다시 배포됩니다.
- 버전 번호: 평소에는 `1.0.x`, 큰 기능이 들어가면 가운데 자리(`1.x.0`)를 올립니다.

BaseLOG_DOCS는 문서만 있어서 `main`으로 바로 PR을 올립니다.

## 커밋과 PR

- 커밋 메시지는 한국어로, 무엇이 바뀌었는지 한 줄로 씁니다. 화면만 바꾸면 `화면: `으로 시작합니다.
  예: `화면: 브라우저 탭 아이콘을 둥지 B 로고로`
- PR 본문은 양식(바뀐 것, 전·후, 확인한 것)을 채웁니다.
- 화면을 바꾸면 **라이트·다크**, **PC(1280px)·휴대폰(375px)** 캡처를 PR에 붙이고, 확인을 받은 뒤 머지합니다.
- 테스트는 건너뛰지 않습니다. 최신 `dev` 기준으로 통과해야 머지합니다.

## 지킬 규칙

- **오류 응답**: `errors`는 항상 목록(없으면 `[]`), 메시지 끝에 마침표를 찍지 않습니다. 요청이 너무 잦으면 429 `TOO_MANY_REQUESTS` "잠시 후 다시 시도해 주세요". 404는 모든 경우 `NOT_FOUND` "볼 수 없는 페이지예요".
- **판정 순서**: 로그인 → 계정 상태 → 볼 수 있나 → 형식·업무 규칙 → 요청 횟수(429는 맨 끝).
- **화면 색**: 화면 코드에 색 값을 직접 쓰지 않습니다. 새 색은 `frontend/src/styles/tokens.css`에 라이트·다크 한 쌍으로 더합니다. `noRawColors.test.ts`가 검사합니다.
- **화면 문구**: "발행" 대신 **"등록"**이라고 씁니다.
- **DB 변경**: Flyway 마이그레이션은 `dev`에 있는 마지막 번호 다음 번호를 씁니다(`backend/src/main/resources/db/migration/`).
- **키와 비밀 값**: `.env`(로컬)와 GitHub Secrets(배포)에만 둡니다. 저장소에 올리지 않습니다.

## 내 컴퓨터에서 실행과 확인

```bash
cp .env.example .env          # 처음 한 번, 키 값 채우기
docker compose up -d --build  # 앱 http://localhost:8080, 메일 http://localhost:8025
```

| 확인 | 명령 |
|---|---|
| 백엔드 테스트 | `cd backend && ./mvnw test` |
| 화면 테스트·검사 | `cd frontend && npm test && npm run lint && npm run format:check` |
| 화면 E2E | `cd frontend && npm run test:e2e` |
| 테스트 계정·글 | `scripts/make-test-accounts.sh`, `scripts/make-test-posts.sh` |
