---
summary: "OpenClaw에서 Ollama (로컬 LLM 런타임) 사용"
read_when:
  - Ollama를 통해 로컬 모델로 OpenClaw을 실행하고 싶을 때
  - Ollama 설정 및 구성 안내가 필요할 때
title: "Ollama"
---

# Ollama

Ollama는 오픈소스 모델을 머신에서 쉽게 실행할 수 있는 로컬 LLM 런타임입니다. OpenClaw은 Ollama의 OpenAI 호환 API와 통합되며, `OLLAMA_API_KEY`로 옵트인하고 명시적 `models.providers.ollama` 항목을 정의하지 않으면 **도구 지원 모델을 자동 발견**할 수 있습니다.

## 빠른 시작

1. Ollama 설치: [https://ollama.ai](https://ollama.ai)

2. 모델 풀:

```bash
ollama pull gpt-oss:20b
# 또는
ollama pull llama3.3
# 또는
ollama pull qwen2.5-coder:32b
```

3. OpenClaw에 Ollama 활성화 (아무 값이나 가능, Ollama는 실제 키가 필요 없음):

```bash
# 환경변수 설정
export OLLAMA_API_KEY="ollama-local"

# 또는 설정 파일에서 구성
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama 모델 사용:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## 모델 발견 (암시적 프로바이더)

`OLLAMA_API_KEY`를 설정하고 `models.providers.ollama`를 정의하지 **않으면**, OpenClaw이 `http://127.0.0.1:11434`의 로컬 Ollama 인스턴스에서 모델을 발견합니다:

- `/api/tags` 및 `/api/show` 쿼리
- `tools` 기능을 보고하는 모델만 유지
- 모델이 `thinking`을 보고할 때 `reasoning` 표시
- 사용 가능한 경우 `contextWindow` 읽기
- 모든 비용을 `0`으로 설정

사용 가능한 모델 확인:

```bash
ollama list
openclaw models list
```

새 모델 추가 - Ollama로 풀하면 자동으로 발견됩니다:

```bash
ollama pull mistral
```

`models.providers.ollama`를 명시적으로 설정하면 자동 발견이 건너뛰어지고 수동으로 모델을 정의해야 합니다.

## 설정

### 기본 설정 (암시적 발견)

가장 간단한 방법:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 명시적 설정 (수동 모델)

다음의 경우 명시적 설정을 사용:

- Ollama가 다른 호스트/포트에서 실행
- 특정 컨텍스트 창이나 모델 목록을 강제하고 싶은 경우
- 도구 지원을 보고하지 않는 모델을 포함하고 싶은 경우

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 81920,
          },
        ],
      },
    },
  },
}
```

### 모델 선택

구성되면 모든 Ollama 모델을 사용할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 고급

### 스트리밍 설정

Ollama의 응답 형식 관련 알려진 문제로 인해, Ollama 모델에 대해 **스트리밍이 기본적으로 비활성화**됩니다. 응답은 한 번에 전달됩니다 (비스트리밍 모드).

스트리밍을 다시 활성화하려면 (도구 지원 모델에서 문제가 발생할 수 있음):

```json5
{
  agents: {
    defaults: {
      models: {
        "ollama/gpt-oss:20b": { streaming: true },
      },
    },
  },
}
```

## 문제 해결

### Ollama가 감지되지 않음

Ollama가 실행 중이고, `OLLAMA_API_KEY`를 설정했으며, 명시적 `models.providers.ollama` 항목을 정의하지 **않았는지** 확인하세요:

```bash
ollama serve
curl http://localhost:11434/api/tags
```

### 사용 가능한 모델 없음

OpenClaw은 도구 지원을 보고하는 모델만 자동 발견합니다. 모델이 나열되지 않으면:

- 도구 지원 모델을 풀하거나
- `models.providers.ollama`에 모델을 명시적으로 정의하세요

### 손상된 응답 또는 출력에 도구 이름 표시

스트리밍 응답의 업스트림 SDK 문제입니다. 최신 OpenClaw 버전에서는 Ollama 모델에 대해 스트리밍을 비활성화하여 **기본적으로 수정**되었습니다.

## 참조

- [모델 프로바이더 개념](/ko-KR/concepts/models) - 모든 프로바이더 개요
- [설정](/ko-KR/gateway/configuration) - 전체 설정 레퍼런스
