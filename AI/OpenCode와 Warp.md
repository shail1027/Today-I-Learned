# OpenCode + oh-my-opencode-slim + Warp 연동

## OpenCode란?

OpenCode는 **터미널에서 동작하는 오픈소스 AI 코딩 에이전트**다. Claude Code처럼 자연어로 코드를 작성/수정/분석하도록 지시할 수 있지만, 결정적인 차이는 **특정 AI 제공자에 종속되지 않는다**는 점이다. Claude, GPT, Gemini, 로컬 모델까지 75개 이상의 LLM 제공자를 지원하며, API 키는 내 로컬 머신에만 저장된다. neovim 사용자 커뮤니티와 terminal.shop 개발진이 만들었으며, "터미널에서 가능한 것의 한계를 밀어붙이겠다"는 철학으로 설계된 툴이다.

## 내부 아키텍처 — 어떻게 동작하는가

OpenCode는 단순한 CLI 래퍼가 아니라 **클라이언트-서버 구조**로 설계되어 있다.

```
[TUI 클라이언트]  <->  [OpenCode 서버 (로컬)]  <->  [LLM API (Claude / GPT / Gemini ...)]
                              │
                        [도구 실행 레이어]
                         파일 읽기/쓰기
                         셸 명령 실행
                         LSP 진단
                         MCP 서버 연결
```

### 요청 흐름

1. 사용자가 TUI에 자연어 프롬프트 입력
2. OpenCode 서버가 프롬프트 + 현재 코드베이스 컨텍스트를 LLM에 전달
3. LLM이 **도구 호출(tool use)** 형태로 응답 — 단순 텍스트가 아니라 "이 파일을 이렇게 수정해라"는 구조화된 명령
4. OpenCode가 도구 호출을 실제 파일 수정, 셸 명령 실행 등으로 실행
5. 결과를 다시 LLM에 피드백 → 필요하면 반복 (agentic loop)

이 구조 덕분에 TUI는 "클라이언트 중 하나"일 뿐이고, 이론적으로 모바일 앱에서 PC의 OpenCode 서버를 원격으로 조종하는 것도 가능하다.

### 내장 도구 (Agent가 쓸 수 있는 것들)

| 도구 | 설명 |
|---|---|
| 파일 읽기/쓰기 | 코드 파일 열람, 수정, 생성 |
| 셸 명령 실행 | 테스트 실행, 빌드, git 등 |
| LSP 연동 | 타입 오류, 심볼 탐색, 레퍼런스 검색 |
| MCP 서버 | 외부 서비스(GitHub, DB 등) 연결 확장 |
| 웹 검색 | 최신 문서 실시간 조회 |

### 두 가지 에이전트 모드

- **build** (기본): 파일 수정, 명령 실행 — 실제 코드 작업 모드
- **plan** (Tab키로 전환): 읽기 전용, 변경 없이 분석·계획만 수립

---

## 설치 방법

```bash
# 가장 간단한 방법 (curl 스크립트)
curl -fsSL https://opencode.ai/install | bash

# npm으로도 설치 가능
npm i -g opencode-ai@latest

# macOS Homebrew (항상 최신 버전)
brew install anomalyco/tap/opencode
```

설치 후 프로젝트 디렉토리에서:

```bash
cd my-project
opencode          # TUI 실행

# TUI 안에서:
/init             # 코드베이스 분석 후 AGENTS.md 자동 생성
/connect          # LLM 제공자 API 키 등록
```

`AGENTS.md`는 OpenCode가 이 프로젝트의 구조와 컨벤션을 이해하기 위한 파일이다. git에 커밋해두면 팀 전체가 같은 컨텍스트를 공유한다.

---

## oh-my-opencode-slim이란?

OpenCode는 기본적으로 하나의 LLM이 모든 작업을 처리한다. `oh-my-opencode-slim`은 이것을 **멀티 에이전트 오케스트레이션 플러그인**으로 확장하는 npm 패키지다.

```bash
npm i -g oh-my-opencode-slim
```

### 핵심 아이디어

"하나의 모델이 모든 걸 잘 할 수는 없다. 역할별로 가장 적합한 모델을 써라."

작업 유형에 따라 다른 에이전트(모델)에게 라우팅한다:

| 에이전트 | 역할 | 권장 모델 |
|---|---|---|
| **Orchestrator** | 전체 작업 계획 및 위임 | 강력한 추론 모델 (GPT-5.5, Claude Opus 등) |
| **Oracle** | 아키텍처 결정, 어려운 디버깅 | 최고 성능 모델 |
| **Librarian** | 문서 검색, 최신 API 조회 | 빠른 저비용 모델 |
| **Explorer** | 코드베이스 스캐닝 | 빠른 저비용 모델 |
| **Designer** | UI/UX, 프론트엔드 | 멀티모달 모델 |
| **Fixer** | 명확히 정의된 구현 작업 | 빠른 저비용 모델 |

### 왜 "slim"인가?

`oh-my-opencode`(원본)의 포크로, 불필요한 기능을 제거하고 **토큰 소비를 대폭 줄인** 경량 버전이다. 오케스트레이션 구조는 그대로 유지하면서 실용성에 집중했다.

### 동작 원리

```
사용자 → Orchestrator (큰 모델)
              │
              ├─ 코드베이스 파악 필요? -> Explorer (작은 모델)
              ├─ 문서 찾기 필요?      -> Librarian (작은 모델)
              ├─ 아키텍처 판단 필요?  -> Oracle (큰 모델)
              ├─ UI 작업 필요?        -> Designer
              └─ 구현 실행?           -> Fixer (작은 모델)
```

Orchestrator가 작업을 분석하고 적절한 서브에이전트에게 위임 → 각 에이전트가 전문 작업 수행 → 결과를 Orchestrator가 종합해 사용자에게 반환.

---

## Warp에서 OpenCode 돌리기

### Warp란?

Warp는 **Rust로 작성된 GPU 렌더링 기반 현대적 터미널**이다. 기본 터미널과 달리 AI 에이전트 워크플로우를 1급 기능으로 지원하는 "Agentic Development Environment(ADE)"를 표방한다.

### OpenCode + Warp 연동 원리

Warp는 OpenCode를 실행하면 자동으로 에이전트 세션을 감지하고 추가 기능을 활성화한다.

```bash
cd my-project
opencode          # Warp 안에서 실행하면 자동 감지
```

활성화되는 기능들:

| 기능 | 설명 |
|---|---|
| **Rich Input** (`Ctrl+G`) | 긴 프롬프트를 위한 전체 텍스트 에디터 모드 |
| **Code Review** | 에이전트가 작성한 코드에 인라인 코멘트 전송 |
| **Agent Notifications** | 작업 완료/입력 필요 시 데스크탑 알림 |
| **Vertical Tab 메타데이터** | 탭바에서 에이전트 세션 상태 모니터링 |
| **Tab Configs** | 세션 설정 저장/복원 |
| **Remote Control** | 세션 링크 공유로 팀원과 협업 |

### Warp 알림 플러그인 설정

```json
// opencode.json (또는 ~/.config/opencode/config.json)
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@warp-dot-dev/opencode-warp"]
}
```

이 플러그인은 **OSC 이스케이프 시퀀스**를 통해 Warp와 통신한다. OpenCode 이벤트 발생 시 플러그인이 특수 문자열을 터미널 출력에 삽입하고, Warp가 이를 감지해 네이티브 알림 UI를 표시하는 구조다.

### 실제 활용

```bash
# 여러 에이전트 병렬 실행 (Vertical Tabs 활용)
# 탭 1: opencode (OpenCode)
# 탭 2: claude (Claude Code)
# 탭 3: codex (OpenAI Codex)
# → 같은 태스크를 주고 결과 비교 가능

# 음성 입력으로 긴 프롬프트 전달 (Warp 음성 전사 기능)
# 이미지 붙여넣기 -> 버그 스크린샷, 디자인 목업 컨텍스트로 전달
```
