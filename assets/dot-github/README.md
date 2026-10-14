<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG/main/frontend/src/assets/logo-b-dark.png" />
  <img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG/main/frontend/src/assets/logo-b-light.png" alt="BaseLOG" width="88" />
</picture>

# BaseLOG

### 생각을 기록하고 나누는 나만의 둥지 | 글쓰기부터 친구와 소통까지

설치 없이 브라우저에서 바로 쓰는 개인 블로그 서비스예요. 내 블로그를 <b>둥지</b>라고 불러요.<br/>
Markdown으로 쓰고, 카테고리로 정리하고, 친구와 댓글·좋아요로 나누고, 매일 들러 둥지의 새를 키워요.

[![사이트](https://img.shields.io/badge/사이트-baselog.java21.net-2f7d3a?style=for-the-badge)](https://baselog.java21.net)

[![CI](https://github.com/AIGJ006/BaseLOG/actions/workflows/ci.yml/badge.svg)](https://github.com/AIGJ006/BaseLOG/actions/workflows/ci.yml)
[![Deploy](https://github.com/AIGJ006/BaseLOG/actions/workflows/deploy.yml/badge.svg)](https://github.com/AIGJ006/BaseLOG/actions/workflows/deploy.yml)
[![Release](https://img.shields.io/github/v/release/AIGJ006/BaseLOG?label=버전&color=2f7d3a)](https://github.com/AIGJ006/BaseLOG/releases)
![Java](https://img.shields.io/badge/Java_21-007396?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_4.1-6DB33F?logo=springboot&logoColor=white)
![React](https://img.shields.io/badge/React_18-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL_18-4169E1?logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)

[🌐 사이트 가기](https://baselog.java21.net) · [💻 코드](https://github.com/AIGJ006/BaseLOG) · [📚 설계 문서](https://github.com/AIGJ006/BaseLOG_DOCS) · [📝 기능 명세](https://github.com/AIGJ006/BaseLOG_DOCS/tree/main/specs) · [🗒️ 업데이트 소식](https://baselog.java21.net/releases)

</div>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/home-dark.png" />
    <img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/home-light.png" alt="BaseLOG 홈 화면 — 최신 글 카드와 요즘 인기 태그" width="880" />
  </picture>
</p>

## 목차

- [왜 BaseLOG인가](#-왜-baselog인가)
- [화면 둘러보기](#-화면-둘러보기)
- [주요 기능](#-주요-기능)
- [아키텍처](#-아키텍처)
- [저장소](#-저장소)
- [기술 스택](#-기술-스택)
- [이렇게 만들었어요](#-이렇게-만들었어요)
- [팀](#-팀)

## 💡 왜 BaseLOG인가

| | 흔한 블로그 | BaseLOG |
| --- | --- | --- |
| ✍️ 글쓰기 | 글자만 입력 | **Markdown + 도구 모음**. 왼쪽에 쓰면 오른쪽에 바로 미리보기, 사진·파일 첨부, 썸네일 고르기 |
| 💾 쓰던 글 | 창을 닫으면 사라짐 | **임시저장**과 브라우저 백업. 다시 열면 이어서 써요 |
| 🗂️ 정리 | 글 목록 하나 | **내 카테고리**(상·하위)와 태그. 둥지 사이드바에서 카테고리별로 모아 봐요 |
| 🏷️ 태그 | 직접 입력 | **AI 태그 추천**. Google Gemini가 막히면 자체 AI(Ollama)로 넘어가요 |
| 🔍 발견 | 최신 글만 | **트렌딩 글·요즘 인기 태그**, 글·사람 검색 |
| 🤝 소통 | 일방향 구독 | **친구**(요청 → 수락)와 친구 피드, 댓글·답글·**비밀 댓글**, 좋아요, 🔔 알림 |
| 🔒 공개 범위 | 공개·비공개 | 공개·**친구 공개**·비공개. 친구 공개 글은 카드에 "👥 친구 공개" 표시 |
| 🐣 꾸준함 | — | **새 키우기**. 매일 들르면 출석, 3일 연속마다 새가 자라고, 모은 포인트로 둥지를 꾸며요 |
| 🌙 보기 | 밝은 화면만 | **다크 모드**(시스템·라이트·다크), PC·휴대폰 화면 |
| 🛡️ 운영 | — | **신고·숨김·정지**, 가입 회원 관리, 공지·업데이트 소식 |

## 📸 화면 둘러보기

<table>
<tr>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/editor-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/editor-light.png" alt="글쓰기 화면" /></picture><br/><b>글쓰기</b> — 도구 모음, 실시간 미리보기, 임시저장, 카테고리 선택</td>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/post-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/post-light.png" alt="글 읽기" /></picture><br/><b>글 읽기</b> — 대표 사진, 코드 강조, 좋아요·댓글</td>
</tr>
<tr>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/blog-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/blog-light.png" alt="내 둥지" /></picture><br/><b>내 둥지</b> — 머리말의 새와 둥지, 카테고리 사이드바, 둥지 안 검색</td>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/notif-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/notif-light.png" alt="알림" /></picture><br/><b>알림</b> — 댓글·좋아요·친구 요청을 바로, 종류별로 끄기</td>
</tr>
<tr>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/search-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/search-light.png" alt="검색" /></picture><br/><b>검색</b> — 글·사람 탭, 관련도순·최신순</td>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/friends-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/friends-light.png" alt="친구 관리" /></picture><br/><b>친구 관리</b> — 친구 요청·친구 목록·친구 피드를 한 화면에</td>
</tr>
<tr>
<td width="50%"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/nest-dark.png" /><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/nest-light.png" alt="새 키우기" /></picture><br/><b>새 키우기</b> — 출석·포인트로 새를 키우고 둥지 꾸미기</td>
<td width="50%"><img src="https://raw.githubusercontent.com/AIGJ006/BaseLOG_DOCS/main/assets/org-profile/home-dark.png" alt="다크 모드 홈" /><br/><b>다크 모드</b> — 설정에서 시스템·라이트·다크 중 고르기</td>
</tr>
</table>

<sub>GitHub를 다크 모드로 보면 캡처도 다크 화면으로 바뀌어요.</sub>

## ✨ 주요 기능

### ✍️ 쓰기
- **Markdown 에디터** — 네이버 블로그처럼 사진·인용구·구분선·링크·코드·표 단추를 누르면 문법이 들어가요.
- **[임시저장] · [글 등록]** — 임시글은 "임시" 탭에, 등록한 글은 "등록" 탭에 모여요.
- **사진·첨부파일** — 글에 사진과 파일을 올리고, 카드에 보일 썸네일을 직접 골라요.
- **AI 태그 추천** — 처음 쓸 때 외부 전송 동의를 받고, 제목과 본문을 보고 태그를 제안해요.
- **공개 범위** — 공개·친구 공개·비공개.
- **내 글 관리** — 등록·임시 탭으로 모아 보고, 지운 글은 휴지통에서 되살려요.

### 📖 읽기와 발견
- **홈** — 최신·트렌딩 탭, 요즘 인기 태그와 그 태그의 인기 글.
- **글 상세** — 대표 사진, 코드 강조, 작성자 칸, 링크 미리보기(Open Graph).
- **태그·검색** — 태그별 글 모아 보기, 글·사람 검색, 둥지 안 검색.

### 🤝 친구와 반응
- **친구** — 둥지 머리말이나 글 작성자 칸에서 [친구 요청], 상대가 수락하면 친구. [친구 관리] 화면에서 요청·목록·피드를 한 번에.
- **좋아요·댓글·답글**, 조회수. **비밀 댓글**은 글쓴이와 댓글 쓴 사람만 봐요.
- **🔔 알림** — 댓글·답글·좋아요·친구 요청·친구 새 글. 종류별로 끌 수 있어요.

### 🐣 새 키우기
- **출석** — 하루 한 번 내 둥지에 들르면 +10점, 7일 연속마다 보너스.
- **자라는 새** — 3일 연속 출석마다 아기 새 → 꼬마 새 → 어린 새 → 어른 새. 오래 안 오면 작아져요.
- **둥지 꾸미기** — 글 등록·댓글·받은 좋아요로 모은 포인트로 배경·둥지·새 장식을 해금해요. 꾸민 둥지는 방문자도 봐요.

### 🔐 계정과 운영
- **가입·로그인** — 이메일(인증 메일), Google·GitHub 계정 중 골라서.
- **프로필·설정** — 사진, 닉네임, 소개, 테마.
- **신고·숨김·정지** — 글·댓글 신고, 관리자 처리 화면과 가입 회원 목록.
- **탈퇴와 복구** — 유예 기간 안에 로그인하면 되살아나요.

## 🏗️ 아키텍처

BaseLOG는 **모듈러 모놀리스**예요. Spring Boot 앱 하나가 React 화면과 API를 함께 내주고, 기능은 모듈로 나눠 이벤트로 이어요(예: 댓글이 달림 → 알림, 좋아요를 받음 → 포인트).

```mermaid
flowchart LR
  U([🧑 브라우저]) -->|HTTPS| N[nginx]
  N -->|blue·green| A
  subgraph A[Spring Boot 앱]
    direction TB
    F[React 화면] ~~~ M
    M[account · post · category · tag<br/>interaction · notification · nest<br/>discovery · media · moderation]
  end
  A --> P[(PostgreSQL)]
  A --> R[(Redis<br/>세션·한도)]
  A --> S[(MinIO<br/>사진·파일)]
  A -.AI 태그.-> G[Gemini / Ollama]
  A -.메일.-> E[Gmail SMTP]
```

PR마다 **GitHub Actions**가 바뀐 쪽만 시험하고(CI), SonarQube가 코드 품질을, CodeRabbit이 리뷰를 남겨요. 릴리스 때 `main`에 합치면 도커 이미지를 만들어 서버에 **blue·green 무중단 배포**해요.

## 📦 저장소

| 저장소 | 내용 |
| --- | --- |
| [**BaseLOG**](https://github.com/AIGJ006/BaseLOG) | 서비스 코드 — `frontend/`(React) · `backend/`(Spring Boot) · `deploy/`(배포 설정) |
| [**BaseLOG_DOCS**](https://github.com/AIGJ006/BaseLOG_DOCS) | 요구사항·설계 문서, 기능별 명세 001~021, constitution |
| [**.github**](https://github.com/AIGJ006/.github) | 이 소개글, 작업 방법(CONTRIBUTING), 공통 PR·이슈 양식 |

## 🛠️ 기술 스택

| 분야 | 사용한 것 |
| --- | --- |
| 화면 | ![React](https://img.shields.io/badge/React_18-61DAFB?logo=react&logoColor=black) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white) |
| 서버 | ![Java](https://img.shields.io/badge/Java_21-007396?logo=openjdk&logoColor=white) ![Spring Boot](https://img.shields.io/badge/Spring_Boot_4.1-6DB33F?logo=springboot&logoColor=white) ![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?logo=springsecurity&logoColor=white) |
| 데이터 | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL_18-4169E1?logo=postgresql&logoColor=white) ![Flyway](https://img.shields.io/badge/Flyway-CC0200?logo=flyway&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white) ![MinIO](https://img.shields.io/badge/MinIO-C72E49?logo=minio&logoColor=white) |
| AI | ![Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?logo=googlegemini&logoColor=white) ![Ollama](https://img.shields.io/badge/Ollama-000000?logo=ollama&logoColor=white) |
| 시험 | ![JUnit](https://img.shields.io/badge/JUnit_5-25A162?logo=junit5&logoColor=white) ![Testcontainers](https://img.shields.io/badge/Testcontainers-291A3F) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?logo=vitest&logoColor=white) ![Playwright](https://img.shields.io/badge/Playwright-2EAD33?logo=playwright&logoColor=white) |
| 품질 | ![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?logo=sonarqubeserver&logoColor=white) ![CodeRabbit](https://img.shields.io/badge/CodeRabbit-FF570A) ![Spotless](https://img.shields.io/badge/Spotless-555555) ![ESLint](https://img.shields.io/badge/ESLint-4B32C3?logo=eslint&logoColor=white) |
| 배포 | ![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=githubactions&logoColor=white) ![nginx](https://img.shields.io/badge/nginx-009639?logo=nginx&logoColor=white) blue·green 무중단 |

## 🧭 이렇게 만들었어요

**스펙 주도 개발**([GitHub Spec Kit](https://github.com/github/spec-kit))로 만들었어요. 기능마다 명세 → 계획 → 작업 목록 → 구현 순서로 진행했어요.

```mermaid
flowchart LR
  A[요구사항 회의] --> B[명세 spec.md] --> C[계획 plan.md] --> D[작업 tasks.md] --> E[feature 브랜치<br/>구현·시험] --> F[dev] --> G[release → main<br/>무중단 배포]
```

- 기능 명세 **21개** — 계정·글쓰기·사진·공개 범위·댓글·태그·좋아요·알림·검색·AI 태그·신고·탈퇴·다크 모드·카테고리·첨부·썸네일·친구·새 키우기
- **Git Flow** — `feature/*` → `dev` → `release/vX.Y.Z` → `main`. 평일 저녁에 릴리스하고, 버전마다 [업데이트 소식](https://baselog.java21.net/releases)을 남겨요(1.0.0 → 1.1.0 친구 → 1.2.0 새 키우기·친구 공개·비밀 댓글)
- 1.0.0 전체 점검 때 시험 **5,200여 건** 통과 (서버 4,183 · 화면 996 · 브라우저 E2E 71)

## 👥 팀

| 이름 | 맡은 일 |
| --- | --- |
| 강성찬 | 요구사항 분석, 명세서 |
| 나민서 | 요구사항 분석, 명세서, 구현·배포 |
| 김민서 | 요구사항 분석, 명세서 |

<p align="center"><sub>© 2026 BaseLOG 팀</sub></p>

---

### 이 저장소

BaseLOG 팀 조직의 공통 설정 저장소입니다. 여기 있는 파일은 조직의 모든 저장소에 기본값으로 쓰입니다.
저장소에 같은 파일이 따로 있으면 그 저장소 것이 먼저입니다.

| 파일 | 쓰이는 곳 |
|---|---|
| [`profile/README.md`](profile/README.md) | 조직 첫 화면 소개글 (https://github.com/AIGJ006) |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | 작업 방법: 저장소 역할, 스펙 → 구현 흐름, 브랜치, 지킬 규칙 |
| [`PULL_REQUEST_TEMPLATE.md`](PULL_REQUEST_TEMPLATE.md) | PR을 열면 본문에 미리 채워지는 양식 |
| [`ISSUE_TEMPLATE/`](ISSUE_TEMPLATE/) | 이슈 양식: 버그 신고, 기능 제안 |

원본은 [AIGJ006/BaseLOG_DOCS](https://github.com/AIGJ006/BaseLOG_DOCS)의 `assets/dot-github/`(소개글은 `assets/org-profile/`)에 있습니다. 고칠 때는 원본을 먼저 바꾸고 여기로 옮겨 붙입니다.
