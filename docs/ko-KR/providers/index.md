---
summary: "OpenClaw에서 지원하는 모델 프로바이더 (LLM)"
read_when:
  - 모델 프로바이더를 선택하고 싶을 때
  - 지원되는 LLM 백엔드 개요가 필요할 때
title: "모델 프로바이더"
---

# 모델 프로바이더

OpenClaw은 많은 LLM 프로바이더를 사용할 수 있습니다. 프로바이더를 선택하고, 인증한 다음, 기본 모델을 `provider/model`로 설정하세요.

채팅 채널 문서 (WhatsApp/Telegram/Discord/Slack 등)를 찾으시나요? [채널](/ko-KR/channels)을 참조하세요.

## 빠른 시작

1. 프로바이더와 인증합니다 (보통 `openclaw onboard`를 통해).
2. 기본 모델을 설정합니다:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 프로바이더 문서

- [Anthropic (API + Claude Code CLI)](/ko-KR/providers/anthropic)
- [OpenAI (API + Codex)](/ko-KR/providers/openai)
- [Ollama (로컬 모델)](/ko-KR/providers/ollama)
- [Google (Gemini)](/ko-KR/providers/google)

### 추가 프로바이더 (영문)

- [OpenRouter](/providers/openrouter)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (Kimi)](/providers/moonshot)
- [Amazon Bedrock](/providers/bedrock)
- [MiniMax](/providers/minimax)
- [Venice (프라이버시 중심)](/providers/venice)
- [Qianfan](/providers/qianfan)
- [GLM 모델](/providers/glm)

## 트랜스크립션 프로바이더

- [Deepgram (오디오 트랜스크립션)](/providers/deepgram)

## 커뮤니티 도구

- [Claude Max API Proxy](/providers/claude-max-api-proxy) - Claude Max/Pro 구독을 OpenAI 호환 API 엔드포인트로 사용

전체 프로바이더 카탈로그와 고급 설정은 [모델 프로바이더 개념](/ko-KR/concepts/models)을 참조하세요.
