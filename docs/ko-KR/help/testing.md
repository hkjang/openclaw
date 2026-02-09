---
summary: "테스트 킷: 단위/E2E/라이브 테스트 스위트와 실행 방법"
read_when:
  - 테스트를 로컬이나 CI에서 실행하고 싶을 때
  - 모델/프로바이더 회귀 테스트를 추가하고 싶을 때
  - Gateway와 에이전트 동작을 디버깅하고 싶을 때
title: "테스팅"
---

# 테스팅

OpenClaw의 테스트 스위트 구성, 실행 방법, 그리고 회귀 테스트 추가 가이드입니다.

## 빠른 시작

```bash
pnpm build && pnpm check && pnpm test   # 전체 게이트 (push 전 권장)
pnpm test:coverage                       # 커버리지 게이트
pnpm test:e2e                            # E2E 스위트
pnpm test:live                           # 라이브 스위트 (실제 자격 증명 필요)
```

## 테스트 스위트

### 단위/통합 테스트 (기본)

| 항목        | 값                        |
| ----------- | ------------------------- |
| **명령어**  | `pnpm test`               |
| **설정**    | `vitest.config.ts`        |
| **파일**    | `src/**/*.test.ts`        |
| **범위**    | 순수 단위 테스트, 인프로세스 통합 테스트, 결정적 회귀 |
| **요구사항** | CI 실행 가능, 실제 키 불필요, 빠르고 안정적 |

### E2E 테스트 (Gateway 스모크)

| 항목        | 값                        |
| ----------- | ------------------------- |
| **명령어**  | `pnpm test:e2e`           |
| **설정**    | `vitest.e2e.config.ts`    |
| **파일**    | `src/**/*.e2e.test.ts`    |
| **범위**    | 멀티 인스턴스 Gateway E2E, WebSocket/HTTP 서피스, 노드 페어링 |
| **요구사항** | CI 실행 가능, 실제 키 불필요, 상대적으로 느림 |

### 라이브 테스트 (실제 프로바이더)

| 항목        | 값                        |
| ----------- | ------------------------- |
| **명령어**  | `pnpm test:live`          |
| **설정**    | `vitest.live.config.ts`   |
| **파일**    | `src/**/*.live.test.ts`   |
| **활성화**  | `OPENCLAW_LIVE_TEST=1` (pnpm test:live가 자동 설정) |
| **범위**    | 실제 자격 증명으로 프로바이더/모델 동작 확인 |
| **요구사항** | CI 불안정 (실제 네트워크), 비용 발생, 좁은 범위 권장 |

## 어떤 스위트를 실행할까?

| 상황                           | 스위트                |
| ------------------------------ | --------------------- |
| 코드 변경 후 빠른 검증         | `pnpm test`           |
| Gateway 통합 확인              | `pnpm test:e2e`       |
| 특정 프로바이더 동작 확인      | `pnpm test:live`      |
| push 전 전체 검증              | `pnpm build && pnpm check && pnpm test` |
| 커버리지 확인                  | `pnpm test:coverage`  |

## 라이브 테스트: 모델 스모크

### 레이어 1: 직접 모델 완성

```bash
# 단일 모델 테스트
OPENCLAW_LIVE_MODELS="openai/gpt-5.2" \
  pnpm test:live src/agents/models.profiles.live.test.ts
```

- **테스트 파일**: `src/agents/models.profiles.live.test.ts`
- **목적**: 발견된 모델을 나열하고, 모델별 소규모 완성 실행
- **활성화**: `OPENCLAW_LIVE_MODELS=modern` (또는 `all`)
- **Modern 허용 목록**: Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4

### 레이어 2: Gateway + Dev 에이전트 스모크

```bash
# 단일 모델, Gateway 스모크
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" \
  pnpm test:live src/gateway/gateway-models.profiles.live.test.ts
```

- **테스트 파일**: `src/gateway/gateway-models.profiles.live.test.ts`
- **목적**: 인프로세스 Gateway 시작, `agent:dev` 세션 생성, 키가 있는 모델 반복
- **검증**: 의미 있는 응답 (도구 없음), 실제 도구 호출
- **프로브 유형**: `read` 프로브, `exec+read` 프로브, 이미지 프로브

### 키 소싱

라이브 테스트는 CLI와 동일한 방식으로 자격 증명을 찾습니다:

| 소스                              | 설명                    |
| --------------------------------- | ----------------------- |
| `~/.openclaw/credentials/`        | 프로필 스토어 (선호)    |
| `~/.openclaw/openclaw.json`       | Config 파일             |
| `~/.profile`                      | 누락된 API 키 폴백     |

**Anthropic 키:**

```bash
OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."   # 여러 키 (콤마 구분)
OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...              # 단일 키
```

## 라이브: Setup-Token 스모크

```bash
OPENCLAW_LIVE_SETUP_TOKEN=1 \
OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test \
OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-... \
  pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## 라이브: CLI 백엔드 스모크

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

| 오버라이드                                 | 기본값                             |
| ------------------------------------------ | ---------------------------------- |
| `OPENCLAW_LIVE_CLI_BACKEND_MODEL`          | `claude-cli/claude-sonnet-4-5`     |
| `OPENCLAW_LIVE_CLI_BACKEND_COMMAND`        | `claude`                           |
| `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE`    | `0`                                |
| `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE`   | `0`                                |

## 권장 라이브 레시피

### 단일 모델 (직접)

```bash
OPENCLAW_LIVE_MODELS="openai/gpt-5.2" \
  pnpm test:live src/agents/models.profiles.live.test.ts
```

### 단일 모델 (Gateway 스모크)

```bash
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" \
  pnpm test:live src/gateway/gateway-models.profiles.live.test.ts
```

### 프로바이더 간 도구 호출

```bash
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" \
  pnpm test:live src/gateway/gateway-models.profiles.live.test.ts
```

### Google 포커스

```bash
# Gemini API
OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live

# Antigravity
OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live
```

## Docker 러너

| 명령어                              | 설명                            |
| ----------------------------------- | ------------------------------- |
| `pnpm test:docker:live-models`      | 직접 모델 테스트                |
| `pnpm test:docker:live-gateway`     | Gateway + dev 에이전트          |
| `pnpm test:docker:onboard`          | 온보딩 마법사 (TTY)             |
| `pnpm test:docker:gateway-network`  | 두 컨테이너, WS 인증 + 헬스    |
| `pnpm test:docker:plugins`          | 커스텀 확장 로드 + 레지스트리   |

### Docker 환경 변수

| 변수                                 | 기본값                   |
| ------------------------------------ | ------------------------ |
| `OPENCLAW_CONFIG_DIR`                | `~/.openclaw`            |
| `OPENCLAW_WORKSPACE_DIR`             | `~/.openclaw/workspace`  |
| `OPENCLAW_PROFILE_FILE`              | `~/.profile`             |
| `OPENCLAW_LIVE_GATEWAY_MODELS`       | —                        |
| `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS` | —                        |

## 문서 검증

```bash
pnpm docs:list
```

문서 파일의 기본 구조와 링크를 확인합니다.

## Deepgram 라이브 테스트

```bash
DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 \
  pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts
```

## 자격 증명 관리

> ⚠️ 자격 증명을 절대 커밋하지 마세요

| 방법                       | 위치                         |
| -------------------------- | ---------------------------- |
| **프로필 스토어** (선호)   | `~/.openclaw/credentials/`   |
| **Config 파일**            | `~/.openclaw/openclaw.json`  |
| **환경 변수**              | `~/.profile`에서 export      |

## 오프라인 회귀 (CI 안전)

CI에서 안전하게 실행할 수 있는 회귀 테스트:

- **모의 도구 호출**: `src/gateway/gateway.tool-calling.mock-openai.test.ts`
- **Gateway 마법사**: `src/gateway/gateway.wizard.e2e.test.ts`

## 회귀 테스트 추가 가이드

### 원칙

1. **CI 안전 회귀 선호**: 모의/스텁 프로바이더로 정확한 요청 변환 캡처
2. **라이브 테스트는 좁은 범위**: opt-in 환경 변수로 제한
3. **가장 작은 레이어 대상**:
   - 프로바이더 요청 변환 → 직접 모델 테스트
   - Gateway 세션/히스토리/도구 파이프라인 → Gateway 라이브 스모크 또는 CI 안전 모의 테스트

### 버그 유형별 테스트 위치

| 버그 유형                    | 테스트 위치                    |
| ---------------------------- | ------------------------------ |
| 프로바이더 API 요청 포맷     | `*.test.ts` (단위)            |
| 도구 호출 파이프라인         | `*.e2e.test.ts` 또는 모의     |
| 특정 모델의 응답 처리        | `*.live.test.ts`              |
| Gateway WebSocket 처리       | `*.e2e.test.ts`               |

## 다음 단계

- [디버깅](/ko-KR/help/debugging) - 디버깅 도구와 기법
- [확장 프로그램 개발](/ko-KR/reference/extension-development) - 플러그인 테스트
- [개발 가이드](/ko-KR/reference/contributing) - 기여 방법
