---
summary: "OpenClaw에서 Google Gemini 모델 사용"
read_when:
  - OpenClaw에서 Google Gemini 모델을 사용하고 싶을 때
  - Gemini API 키 설정이 필요할 때
title: "Google (Gemini)"
---

# Google (Gemini)

Google은 **Gemini** 모델 패밀리를 제공합니다. OpenClaw에서 API 키로 인증할 수 있습니다.

## API 키 설정

1. [Google AI Studio](https://aistudio.google.com/)에서 API 키를 생성합니다.
2. 환경변수 또는 설정 파일에 키를 추가합니다.

### CLI 설정

```bash
openclaw onboard --auth-choice google-api-key
```

### 설정 스니펫

```json5
{
  env: { GEMINI_API_KEY: "..." },
  agents: {
    defaults: {
      model: { primary: "google/gemini-3-flash-preview" },
    },
  },
}
```

## 사용 가능한 모델

| 모델 | 설명 |
|------|------|
| `google/gemini-3-flash-preview` | 빠르고 효율적인 모델 |
| `google/gemini-3-pro-preview` | 고성능 모델 |

## 미디어 처리

Google Gemini 모델은 비디오 분석에 사용할 수 있습니다:

```json5
{
  tools: {
    media: {
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

## 임베딩 (메모리 검색)

Gemini 임베딩 모델을 메모리 검색에 사용할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "gemini",
        model: "gemini-embedding-001",
        remote: {
          apiKey: "${GEMINI_API_KEY}",
        },
      },
    },
  },
}
```

## OAuth 인증 (Google Antigravity)

API 키 대신 Google OAuth를 사용하려면 `google-antigravity-auth` 플러그인을 활성화하세요:

```json5
{
  plugins: {
    entries: {
      "google-antigravity-auth": { enabled: true },
    },
  },
}
```

그런 다음:

```bash
openclaw models auth login --provider google
```

## 참고

- 모델 참조는 항상 `provider/model`을 사용합니다.
- 인증 세부사항은 [인증](/ko-KR/gateway/authentication)을 참조하세요.

## 문제 해결

**API 키 오류**

- API 키가 올바른지 확인하세요: `openclaw models status`
- Google AI Studio에서 키를 다시 생성하세요

**모델을 찾을 수 없음**

- `openclaw models list`로 사용 가능한 모델을 확인하세요
- 프로바이더 ID가 `google`인지 확인하세요
