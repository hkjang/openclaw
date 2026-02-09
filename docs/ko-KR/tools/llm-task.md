---
summary: "JSON 전용 LLM 작업 플러그인으로 스키마 기반 자동화"
read_when:
  - 에이전트가 구조화된 JSON 작업을 처리하게 하고 싶을 때
  - 워크플로우 자동화에 LLM을 활용하고 싶을 때
title: "LLM 작업 (llm-task)"
---

# LLM 작업 (llm-task)

`llm-task`는 JSON Schema로 검증된 구조화 출력을 생성하는 선택적 플러그인 도구입니다. 에이전트의 도구 없이 순수 LLM 추론만 사용합니다.

## 활성화

```json5
{
  plugins: {
    entries: {
      "llm-task": {
        enabled: true,
        config: {
          defaultProvider: "openai-codex",
          defaultModel: "gpt-5.2",
          allowedModels: ["openai-codex/gpt-5.3-codex"],
          maxTokens: 800,
          timeoutMs: 30000,
        },
      },
    },
  },
}
```

## 도구 파라미터

| 파라미터          | 필수 | 설명                             |
| ----------------- | ---- | -------------------------------- |
| `prompt`          | 예   | LLM에 보낼 프롬프트              |
| `input`           | 아니오 | 추가 입력 데이터               |
| `schema`          | 아니오 | 출력 JSON Schema              |
| `provider`        | 아니오 | 프로바이더 지정                |
| `model`           | 아니오 | 모델 지정                      |
| `authProfileId`   | 아니오 | 인증 프로필 ID                 |
| `temperature`     | 아니오 | 온도 설정                      |
| `maxTokens`       | 아니오 | 최대 토큰                      |
| `timeoutMs`       | 아니오 | 타임아웃 (ms)                  |

## 사용 예시

에이전트가 자동으로 `llm_task` 도구를 호출합니다:

```json
{
  "tool": "llm_task",
  "args": {
    "prompt": "다음 텍스트에서 감정을 분석하세요",
    "input": "오늘 날씨가 정말 좋아서 기분이 좋습니다",
    "schema": {
      "type": "object",
      "properties": {
        "sentiment": { "enum": ["positive", "negative", "neutral"] },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
      },
      "required": ["sentiment", "confidence"]
    }
  }
}
```

결과:

```json
{
  "sentiment": "positive",
  "confidence": 0.95
}
```

## 안전성

- **JSON 전용**: 도구 사용 없이 순수 텍스트 생성만
- **스키마 검증**: 출력이 JSON Schema를 통과해야 유효
- **격리**: 메인 에이전트와 다른 모델/프로바이더 사용 가능
- **신뢰하지 않기**: 스키마 검증을 통과해도 출력을 맹목적으로 신뢰하지 마세요

## 다음 단계

- [도구 개요](/ko-KR/tools) - 에이전트 도구 시스템
- [플러그인](/ko-KR/tools/plugins) - 플러그인 관리
