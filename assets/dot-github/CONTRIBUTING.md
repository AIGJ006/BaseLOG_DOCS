# BaseLOG 작업 방법

BaseLOG(https://baselog.java21.net)는 강성찬·나민서·김민서가 만드는 티스토리 같은 블로그 서비스입니다. 화면에서는 개인 블로그를 **둥지**라고 부릅니다.
이 문서는 팀 저장소 전체에 적용되는 작업 방법입니다.

## 저장소

| 저장소 | 내용 |
|---|---|
| [BaseLOG](https://github.com/AIGJ006/BaseLOG) | 구현 코드: `backend/`(Spring Boot + PostgreSQL + Flyway), `frontend/`(React), `deploy/`, CI·배포 workflow. 개발 환경은 [DEVELOPMENT.md](https://github.com/AIGJ006/BaseLOG/blob/main/DEVELOPMENT.md) |
| [BaseLOG_DOCS](https://github.com/AIGJ006/BaseLOG_DOCS) | 설계 문서: 원문 `docs/01~51`, 기능별 `specs/001~021`, 원칙 `.specify/memory/constitution.md`, Spec Kit 명령 |
| [.github](https://github.com/AIGJ006/.github) | 조직 소개글, 이 문서, 공통 PR·이슈 양식 |

코드 주석의 `docs/NN`, `specs/NNN-기능` 경로는 BaseLOG_DOCS 기준입니다.

## 새 기능은 스펙부터

[GitHub Spec Kit](https://github.com/github/spec-kit)으로 스펙을 먼저 쓰고 구현합니다.

1. **BaseLOG_DOCS**에서 `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks`로 `specs/NNN-기능/`을 만듭니다.
2. 팀과 정한 것은 BaseLOG_DOCS README의 **"정해진 것"** 표에 날짜와 함께 적습니다. 공통 스펙(001~016)을 바꿀 때는 팀에 먼저 묻습니다. 017 이후(카테고리·첨부·썸네일·친구·새 키우기)는 나민서 개인 확장입니다.
3. **BaseLOG**에서 `/speckit-implement`로 tasks를 따라 구현합니다.
4. API 계약(`openapi.yaml`)을 바꾸면 BaseLOG의 `backend/src/test/resources/contracts/<기능>/openapi.yaml` 복사본도 같이 바꿉니다. 계약 시험이 이 복사본을 읽습니다.

## 브랜치와 배포 (BaseLOG, Git Flow)

```
feature/*  ──PR──▶  dev  ──(평일 17:45 릴리스)──▶ release/vX.Y.Z ──PR──▶ main
                     ▲                                                  │
hotfix/*  ──PR──────┴──────────────────────────────────────────PR───────┘
```

| 브랜치 | 만드는 곳 → 합치는 곳 | 하는 일 |
|---|---|---|
| `main` | — | 실제 사이트 버전. **머지하면 서버에 blue·green 무중단 배포**됩니다 |
| `dev` | — | 개발 브랜치. 서버에는 올라가지 않고 로컬에서 확인합니다 |
| `feature/<이름>` | dev → dev (PR) | 기능·화면·버그 수정 하나. 예: `feature/friend-visibility` |
| `release/vX.Y.Z` | dev → main (PR) | 릴리스 노트 `frontend/src/features/site-news/releases/vX.Y.Z.md`를 넣어 main에 올립니다. 머지하면 GitHub 릴리스와 사이트 "업데이트 소식"이 생깁니다 |
| `hotfix/<이름>` | main → dev와 main (PR 두 개) | 사이트가 안 열리는 등 급한 고침 |

- **버전**: 평소 수정은 끝자리(1.2.0 → 1.2.1), 큰 기능은 가운데 자리(1.2 → 1.3). 평일 17:45에 릴리스하고, 주말 수정분은 월요일에 묶습니다.
- **되돌리기**: `main`에서 릴리스 머지를 revert하는 PR을 머지하면 직전 버전으로 다시 배포됩니다. 릴리스 태그를 골라 Actions → deploy → Run workflow로 다시 배포할 수도 있습니다.

BaseLOG_DOCS는 문서만 있어서 `main`으로 바로 PR을 올립니다.

## 커밋과 PR

- 커밋 메시지는 한국어로, 무엇이 바뀌었는지 한 줄로 씁니다. 앞에 갈래를 붙이면 찾기 쉽습니다: `화면:`, `feat(nest):`, `fix:`, `CI:`, `배포:`.
  예: `화면: 브라우저 탭 아이콘을 둥지 B 로고로`
- PR 본문은 양식(바뀐 것, 전·후, 확인한 것)을 채웁니다.
- 화면을 바꾸면 **라이트·다크**, **PC(1280px)·휴대폰(375px)** 캡처를 PR에 붙이고, 확인을 받은 뒤 머지합니다.
- PR마다 **CI**(`ci.yml`: 백엔드 `./mvnw verify`, 화면 lint·test·build, 바뀐 쪽만)가 초록이어야 머지합니다. 테스트는 건너뛰지 않습니다.
- **SonarQube**(코드 품질·커버리지)와 **CodeRabbit**(한국어 AI 리뷰)이 PR에 결과를 남깁니다. 머지를 막지는 않지만 신뢰성·보안 지적은 고칩니다.

## 지킬 규칙

- **오류 응답**: `errors`는 항상 목록(없으면 `[]`), 메시지 끝에 마침표를 찍지 않습니다. 요청이 너무 잦으면 429 `TOO_MANY_REQUESTS` "잠시 후 다시 시도해 주세요". 404는 모든 경우 `NOT_FOUND` "볼 수 없는 페이지예요".
- **판정 순서**: 로그인 → 계정 상태 → 볼 수 있나 → 형식·업무 규칙 → 요청 횟수(429는 맨 끝).
- **화면 색**: 화면 코드에 색 값을 직접 쓰지 않습니다. 새 색은 `frontend/src/styles/tokens.css`에 라이트·다크 한 쌍으로 더합니다. `noRawColors.test.ts`가 검사합니다.
- **화면 문구**: "발행" 대신 **"등록"**, "블로그" 대신 **"둥지"**(내 둥지, 둥지 주소)라고 씁니다.
- **DB 변경**: Flyway 마이그레이션은 `dev`에 있는 마지막 번호 다음 번호를 씁니다(`backend/src/main/resources/db/migration/`). 무중단 배포 중에는 옛 앱과 새 앱이 같은 DB를 쓰므로, 칸·테이블은 **더하기만** 하고 지우기·이름 바꾸기는 다음 릴리스로 미룹니다.
- **코드 모양**: 백엔드는 `./mvnw spotless:apply`, 화면은 `npm run format`으로 맞춥니다.
- **키와 비밀 값**: `.env`(로컬)와 GitHub Secrets(배포)에만 둡니다. 저장소에 올리지 않습니다.

## 내 컴퓨터에서 실행과 확인

```bash
cp .env.example .env          # 처음 한 번, 키 값 채우기
docker compose up -d --build  # 앱 http://localhost:8080, 메일 http://localhost:8025
```

| 확인 | 명령 |
|---|---|
| 백엔드 시험 | `cd backend && ./mvnw verify` (Docker 필요, Testcontainers) |
| 화면 시험·검사 | `cd frontend && npm test && npm run lint && npm run build` |
| 화면 E2E | `cd frontend && npm run test:e2e` |
| 화면만 빠르게 | `cd frontend && npm run dev` (http://localhost:5173) |
| 테스트 계정·글 | `sh scripts/make-test-accounts.sh` → `sh scripts/make-test-posts.sh` |
