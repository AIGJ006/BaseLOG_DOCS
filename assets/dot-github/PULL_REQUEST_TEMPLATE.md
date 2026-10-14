## 무엇이 바뀌나요

<!-- 관련 스펙이 있으면 적어 주세요. 예: specs/021-nest-decor FR-012 -->

Before:

After:

## 왜

## 확인한 것

- [ ] PR 대상 브랜치가 맞아요 (BaseLOG는 `feature/*` → `dev`, 급한 고침만 `hotfix/*` → `main`)
- [ ] 백엔드를 바꿨다면 `cd backend && ./mvnw spotless:check` (틀리면 `spotless:apply`)
- [ ] 화면을 바꿨다면 `cd frontend && npm run lint && npm test`, 라이트·다크 캡처를 붙임
- [ ] DB 마이그레이션(Flyway)이 있으면 더하기만 했음 (옛 앱이 같은 DB로 계속 돌 수 있게)
- [ ] API 계약이나 팀과 정한 것이 바뀌면 BaseLOG_DOCS 스펙·"정해진 것" 표와 `contracts/` 복사본도 같이 바꿈
