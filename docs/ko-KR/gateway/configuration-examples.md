---
summary: "일반적인 OpenClaw 설정을 위한 스키마 정확한 설정 예시"
read_when:
  - OpenClaw 설정 방법을 배울 때
  - 설정 예시를 찾을 때
  - OpenClaw을 처음 설정할 때
title: "설정 예시"
---

# 설정 예시

아래 예시는 현재 설정 스키마에 맞추어져 있습니다. 전체 레퍼런스와 필드별 설명은 [설정 가이드](/ko-KR/gateway/configuration)를 참조하세요.

## 빠른 시작

### 절대 최소 설정

```json5
{
  agent: { workspace: "~/.openclaw/workspace" },
  channels: { whatsapp: { allowFrom: ["+821012345678"] } },
}
```

`~/.openclaw/openclaw.json`에 저장하면 해당 번호에서 봇에 DM을 보낼 수 있습니다.

### 권장 시작 설정

```json5
{
  identity: {
    name: "Clawd",
    theme: "helpful assistant",
    emoji: "🦞",
  },
  agent: {
    workspace: "~/.openclaw/workspace",
    model: { primary: "anthropic/claude-sonnet-4-5" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+821012345678"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## 확장 예시 (주요 옵션)

> JSON5는 주석과 후행 쉼표를 사용할 수 있습니다. 일반 JSON도 작동합니다.

```json5
{
  // 환경변수 + 셸
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },

  // 인증 프로필 메타데이터 (비밀키는 auth-profiles.json에)
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
      "openai:default": { provider: "openai", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
      openai: ["openai:default"],
    },
  },

  // 아이덴티티
  identity: { name: "Samantha", theme: "helpful sloth", emoji: "🦥" },

  // 로깅
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
  },

  // 메시지 포맷팅
  messages: {
    messagePrefix: "[openclaw]",
    responsePrefix: ">",
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
  },

  // 라우팅 + 큐
  routing: {
    groupChat: { mentionPatterns: ["@openclaw", "openclaw"], historyLimit: 50 },
    queue: { mode: "collect", debounceMs: 1000, cap: 20, drop: "summarize" },
  },

  // 세션 동작
  session: {
    scope: "per-sender",
    reset: { mode: "daily", atHour: 4, idleMinutes: 60 },
    resetTriggers: ["/new", "/reset"],
  },

  // 채널 설정
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+821012345678"],
      groups: { "*": { requireMention: true } },
    },
    telegram: {
      enabled: true,
      botToken: "YOUR_TELEGRAM_BOT_TOKEN",
      allowFrom: ["123456789"],
      groups: { "*": { requireMention: true } },
    },
    discord: {
      enabled: true,
      token: "YOUR_DISCORD_BOT_TOKEN",
      dm: { enabled: true, allowFrom: ["username"] },
    },
    slack: {
      enabled: true,
      botToken: "xoxb-REPLACE_ME",
      appToken: "xapp-REPLACE_ME",
      dm: { enabled: true, allowFrom: ["U123"] },
    },
  },

  // 에이전트 런타임
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      userTimezone: "Asia/Seoul",
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["anthropic/claude-opus-4-6", "openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "openai/gpt-5.2": { alias: "gpt" },
      },
      thinkingDefault: "low",
      maxConcurrent: 3,
      sandbox: { mode: "non-main" },
    },
  },

  // Gateway + 네트워킹
  gateway: {
    mode: "local",
    port: 18789,
    bind: "loopback",
    controlUi: { enabled: true, basePath: "/openclaw" },
    auth: { mode: "token", token: "gateway-token" },
  },
}
```

## 일반적인 패턴

### 멀티 플랫폼 설정

```json5
{
  agent: { workspace: "~/.openclaw/workspace" },
  channels: {
    whatsapp: { allowFrom: ["+821012345678"] },
    telegram: {
      enabled: true,
      botToken: "YOUR_TOKEN",
      allowFrom: ["123456789"],
    },
    discord: {
      enabled: true,
      token: "YOUR_TOKEN",
      dm: { allowFrom: ["yourname"] },
    },
  },
}
```

### 안전한 DM 모드 (공유 인박스 / 멀티 유저 DM)

여러 사람이 봇에 DM을 보낼 수 있는 경우, **안전한 DM 모드**를 활성화하여 서로 다른 발신자의 DM이 하나의 컨텍스트를 공유하지 않도록 합니다:

```json5
{
  session: { dmScope: "per-channel-peer" },
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+821012345678", "+821087654321"],
    },
    discord: {
      enabled: true,
      token: "YOUR_DISCORD_BOT_TOKEN",
      dm: { enabled: true, allowFrom: ["alice", "bob"] },
    },
  },
}
```

### OAuth + API 키 폴오버

```json5
{
  auth: {
    profiles: {
      "anthropic:subscription": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:api": { provider: "anthropic", mode: "api_key" },
    },
    order: { anthropic: ["anthropic:subscription", "anthropic:api"] },
  },
  agent: {
    workspace: "~/.openclaw/workspace",
    model: { primary: "anthropic/claude-sonnet-4-5", fallbacks: ["anthropic/claude-opus-4-6"] },
  },
}
```

### 업무용 봇 (접근 제한)

```json5
{
  identity: { name: "WorkBot", theme: "professional assistant" },
  agent: { workspace: "~/work-openclaw", elevated: { enabled: false } },
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      channels: {
        "#engineering": { allow: true, requireMention: true },
        "#general": { allow: true, requireMention: true },
      },
    },
  },
}
```

### 로컬 모델 전용

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
    model: { primary: "lmstudio/minimax-m2.1-gs32" },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 팁

- `dmPolicy: "open"`을 설정하면 해당 `allowFrom` 목록에 `"*"`를 포함해야 합니다.
- 프로바이더 ID는 다릅니다 (전화번호, 사용자 ID, 채널 ID). 프로바이더 문서에서 형식을 확인하세요.
- 나중에 추가할 수 있는 선택적 섹션: `web`, `browser`, `ui`, `discovery`, `canvasHost`, `talk`, `signal`, `imessage`.
- 자세한 설정은 [문제 해결](/ko-KR/gateway/gateway-troubleshooting)을 참조하세요.
