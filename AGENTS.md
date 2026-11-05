# premiere-pro-mcp — Adobe Premiere Pro를 MCP로 조작하는 브리지

## 왜 있나

- 누구를 위한 무엇인가: Codex·Claude Code·Claude Desktop 등 MCP 클라이언트가 로컬 Premiere Pro 프로젝트를 편집하게 하는 서버(CEP 패널이 지원 브리지, UXP는 실험적). README 기준 도구 283개를 `search_tools`/`invoke_tool` 뒤에 둔다.
- 지금 상태: 운영 중(배포 중). package.json 버전 1.2.8, npm `adobe-premiere-pro-mcp`. 웹사이트 `website/` 별도 포함.

## 구조

```
src/            MCP 서버 (cli.ts·index.ts, bridge/ tools/ prompts/ resources/, __tests__/)
cep-plugin/     Premiere 패널 브리지 (bridge-cep.js 등)
uxp-plugin/     실험적 읽기 전용 어댑터
scripts/        설치·doctor(mac/win)·verify-package·build-mcpb·sign-zxp·live-tool-sweep
skills/ plugins/ claude-plugins/ .agents/  Codex·Claude 플러그인과 스킬
website/        premiere-mcp.com (vite + Cloudflare Worker, wrangler.jsonc)
.github/workflows/  sign-zxp, upload-release-assets, windows-setup
```

## 명령

|할 일|명령|
|---|---|
|설치|`npm install`, 전체 로컬 설치 `npm run setup:mac` / `npm run setup:win`|
|개발 실행|`npm run dev`, `npm run start`|
|테스트|`npm test -- --runInBand`, 실기 점검 `node scripts/live-tool-sweep.mjs` (스크래치 프로젝트에서)|
|빌드·점검|`npm run build`, `npm run lint`, `npm run pack:check`, `npm run doctor`|
|배포|npm 게시 명령 미확인(`prepack` 이 빌드). 웹사이트: `website/` 에서 `npm run deploy` (vitest → vite build → wrangler deploy)|

## 배포와 확인

- 어디에 도나: npm 패키지, GitHub Release 자산(CEP 아카이브, upload-release-assets.yml), 웹사이트 `premiere-mcp.com`·`www.premiere-mcp.com`(Cloudflare Worker `premiere-mcp-web`).
- 자동/수동: Release `published` 시 워크플로가 CEP 자산을 올린다. npm 게시용 GitHub Actions 워크플로는 실패로 제거됐다(커밋 2394ace) → npm 게시는 수동으로 추정, 미확인. 웹사이트는 수동 `npm run deploy`.
- 배포 확인: `premiere-pro-mcp --doctor`, 설치 후 `verify_premiere_connection`. 원격 버전 확인 명령은 미확인.

## 이 저장소에서 승인이 필요한 일

- npm 게시·GitHub Release 발행: 공개 패키지 버전은 되돌리기 어렵고 CEP 자산 업로드가 자동 실행된다.
- `website/` Cloudflare 배포: 운영 도메인 premiere-mcp.com 이 바뀐다.
- ZXP 서명(`sign:zxp`, sign-zxp.yml): 서명 인증서를 사용한다.
- 실제 Premiere 프로젝트를 대상으로 한 편집·`live-tool-sweep`: 사용자 작업 프로젝트를 변경할 수 있다.

## 하지 않는 것

- 실제 편집을 한다고 주장하는 도구를 스키마 검증만 하고 내놓지 않는다: CONTRIBUTING이 명시한다.
- 도구 개수를 문서에서 가져와 쓰지 않는다: KNOWN_ISSUES가 낡은 수치(104)를 지적했다. 빌드의 `getAvailableTools().length` 로 센다.
- 빌드·테스트가 있는 변경에서 `cep-plugin/bridge-cep.js` 수정 후 패널 재로드를 빼먹지 않는다: CONTRIBUTING 절차.
- 현재 미커밋 변경이 있다. 임의로 되돌리거나 커밋하지 않는다.

## 사고 이력

- CEP evalScript 타임아웃이 두 번째 네이티브 호출을 시작했다(#87 수정) → 타임아웃 후 중복 호출을 막는다.
- Premiere 2025 이후에서 CEP 패널이 안 떴고 2023·2024 밖 설치를 못 찾았다 → 버전 감지를 확장했다.
- npm 게시 GitHub Actions 워크플로가 실패해 제거됐다(2394ace).
- `move_clip_to_track` 이 대상 클립을 덮어썼다 → 대상 클립을 치우고 맞춘 뒤 이동하도록 바꿨다(edc0b0f).
