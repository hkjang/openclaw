---
summary: "OpenClaw에서 Anthropic Claude를 API 키 또는 setup-token으로 사용"
read_when:
  - OpenClaw에서 Anthropic 모델을 사용하고 싶을 때
  - API 키 대신 setup-token을 사용하고 싶을 때
title: "Anthropic"
---

# Anthropic (Claude)

Anthropic은 **Claude** 모델 패밀리를 빌드하고 API를 통해 접근을 제공합니다. OpenClaw에서는 API 키 또는 **setup-token**으로 인증할 수 있습니다.

## 옵션 A: Anthropic API 키

**적합한 경우:** 표준 API 접근 및 사용량 기반 과금.
Anthropic Console에서 API 키를 생성하세요.

### CLI 설정

```bash
openclaw onboard
# 선택: Anthropic API key

# 또는 비대화형
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 설정 스니펫

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 프롬프트 캐싱 (Anthropic API)

OpenClaw은 Anthropic의 프롬프트 캐싱 기능을 지원합니다. 이것은 **API 전용**이며, 구독 인증은 캐시 설정을 적용하지 않습니다.

### 설정

모델 설정에서 `cacheRetention` 파라미터를 사용합니다:

| 값 | 캐시 기간 | 설명 |
|---|---------|------|
| `none` | 캐싱 없음 | 프롬프트 캐싱 비활성화 |
| `short` | 5분 | API 키 인증의 기본값 |
| `long` | 1시간 | 확장 캐시 (베타 플래그 필요) |

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

## 옵션 B: Claude setup-token

**적합한 경우:** Claude 구독 사용.

### setup-token 얻는 방법

setup-token은 Anthropic Console이 아닌 **Claude Code CLI**에서 생성됩니다. **아무 머신**에서 실행할 수 있습니다:

```bash
claude setup-token
```

OpenClaw에 토큰 붙여넣기 (위저드: **Anthropic token (paste setup-token)**), 또는 Gateway 호스트에서 실행:

```bash
openclaw models auth setup-token --provider anthropic
```

다른 머신에서 토큰을 생성한 경우:

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI 설정 (setup-token)

```bash
# 온보딩 중 setup-token 붙여넣기
openclaw onboard --auth-choice setup-token
```

## 참고

- `claude setup-token`으로 setup-token을 생성하고 붙여넣거나, Gateway 호스트에서 `openclaw models auth setup-token`을 실행하세요.
- Claude 구독에서 "OAuth token refresh failed ..." 오류가 표시되면 setup-token으로 재인증하세요.
- 인증 세부사항 + 재사용 규칙은 [OAuth 개념](/ko-KR/gateway/oauth)에 있습니다.

## 문제 해결

**401 오류 / 토큰 갑자기 무효**

- Claude 구독 인증이 만료되거나 취소될 수 있습니다. `claude setup-token`을 다시 실행하고 **Gateway 호스트**에 붙여넣으세요.

**프로바이더 "anthropic"에 대한 API 키를 찾을 수 없음**

- 인증은 **에이전트별**입니다. 새 에이전트는 메인 에이전트의 키를 상속하지 않습니다.
- 해당 에이전트에 대해 온보딩을 다시 실행하거나 setup-token / API 키를 붙여넣으세요.

**프로필 `anthropic:default`에 대한 자격 증명을 찾을 수 없음**

- `openclaw models status`를 실행하여 활성 인증 프로필을 확인하세요.

**사용 가능한 인증 프로필 없음 (모두 쿨다운/사용 불가)**

- `openclaw models status --json`에서 `auth.unusableProfiles`를 확인하세요.
- 다른 Anthropic 프로필을 추가하거나 쿨다운을 기다리세요.

자세한 내용: [문제 해결](/ko-KR/gateway/gateway-troubleshooting) 및 [FAQ](/ko-KR/help/faq).
