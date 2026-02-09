---
summary: "모델 인증: OAuth, API 키, setup-token 설정 가이드"
read_when:
  - 모델 인증이나 OAuth 만료를 디버깅할 때
  - 인증 또는 자격 증명 저장소를 문서화할 때
title: "인증"
---

# 인증

OpenClaw은 모델 프로바이더를 위한 OAuth 및 API 키를 지원합니다. Anthropic 계정의 경우 **API 키** 사용을 권장합니다. Claude 구독 접근의 경우 `claude setup-token`으로 생성된 장기 토큰을 사용하세요.

전체 OAuth 흐름 및 저장소 구조는 [OAuth 개념](/ko-KR/gateway/oauth)을 참조하세요.

## 권장: Anthropic API 키 설정

Anthropic을 직접 사용하는 경우 API 키를 사용하세요.

1. Anthropic Console에서 API 키를 생성합니다.
2. **Gateway 호스트** (즉, `openclaw gateway`를 실행하는 머신)에 키를 설정합니다.

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Gateway가 systemd/launchd 아래에서 실행되는 경우, 데몬이 읽을 수 있도록 `~/.openclaw/.env`에 키를 넣는 것이 좋습니다:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

그런 다음 데몬을 재시작하고 확인합니다:

```bash
openclaw models status
openclaw doctor
```

환경변수를 직접 관리하고 싶지 않다면, 온보딩 위저드가 데몬용 API 키를 저장할 수 있습니다: `openclaw onboard`.

## Anthropic: setup-token (구독 인증)

Anthropic의 경우 **API 키**를 권장합니다. Claude 구독을 사용하는 경우 setup-token 흐름도 지원됩니다. **Gateway 호스트**에서 실행하세요:

```bash
claude setup-token
```

그런 다음 OpenClaw에 붙여넣기:

```bash
openclaw models auth setup-token --provider anthropic
```

다른 머신에서 토큰을 생성한 경우 수동으로 붙여넣기:

```bash
openclaw models auth paste-token --provider anthropic
```

다음과 같은 Anthropic 오류가 표시되면:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

대신 Anthropic API 키를 사용하세요.

수동 토큰 입력 (모든 프로바이더, `auth-profiles.json` 작성 + 설정 업데이트):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

자동화 친화적인 확인 (만료/누락 시 `exit 1`, 곧 만료 시 `exit 2`):

```bash
openclaw models status --check
```

> `claude setup-token`은 대화형 TTY가 필요합니다.

## 인증 상태 확인

```bash
openclaw models status
openclaw doctor
```

## 사용할 자격 증명 제어

### 세션별 (chat 명령)

`/model <alias-또는-id>@<profileId>`로 현재 세션에 특정 프로바이더 자격 증명을 고정합니다 (예: `anthropic:default`, `anthropic:work`).

`/model` (또는 `/model list`)로 간단한 선택기를 표시하고, `/model status`로 전체 보기를 표시합니다 (후보 + 다음 인증 프로필, 그리고 구성된 경우 프로바이더 엔드포인트 세부 정보).

### 에이전트별 (CLI 오버라이드)

에이전트에 대한 명시적 인증 프로필 순서 오버라이드를 설정합니다 (해당 에이전트의 `auth-profiles.json`에 저장):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

특정 에이전트를 대상으로 하려면 `--agent <id>`를 사용하고, 생략하면 구성된 기본 에이전트를 사용합니다.

## 문제 해결

### "No credentials found"

Anthropic 토큰 프로필이 없는 경우 **Gateway 호스트**에서 `claude setup-token`을 실행한 다음 다시 확인:

```bash
openclaw models status
```

### 토큰 만료/만료됨

`openclaw models status`를 실행하여 어떤 프로필이 만료되는지 확인합니다. 프로필이 없으면 `claude setup-token`을 다시 실행하고 토큰을 다시 붙여넣으세요.

## 요구사항

- Claude Max 또는 Pro 구독 (`claude setup-token` 사용 시)
- Claude Code CLI 설치됨 (`claude` 명령 사용 가능)
