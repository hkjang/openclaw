---
summary: "OpenClaw 샌드박싱 작동 방식: 모드, 범위, 워크스페이스 접근, 이미지"
read_when:
  - 샌드박싱에 대한 전용 설명이 필요할 때
  - agents.defaults.sandbox를 조정해야 할 때
title: "샌드박싱"
---

# 샌드박싱

OpenClaw은 피해 범위를 줄이기 위해 **Docker 컨테이너 내에서 도구를 실행**할 수 있습니다. 이것은 **선택 사항**이며 설정(`agents.defaults.sandbox` 또는 `agents.list[].sandbox`)으로 제어됩니다. 샌드박싱이 꺼져 있으면 도구는 호스트에서 실행됩니다.

이것은 완벽한 보안 경계가 아니지만, 모델이 실수할 때 파일시스템과 프로세스 접근을 실질적으로 제한합니다.

## 샌드박스화되는 것

- 도구 실행 (`exec`, `read`, `write`, `edit`, `apply_patch`, `process` 등)
- 선택적 샌드박스 브라우저 (`agents.defaults.sandbox.browser`)

샌드박스화되지 않는 것:

- Gateway 프로세스 자체
- 호스트에서 실행이 명시적으로 허용된 도구 (예: `tools.elevated`)
  - **Elevated exec는 호스트에서 실행되며 샌드박싱을 우회합니다.**

## 모드

`agents.defaults.sandbox.mode`는 샌드박싱이 **언제** 사용되는지 제어합니다:

- `"off"`: 샌드박싱 없음
- `"non-main"`: **비-메인** 세션만 샌드박스 (기본 채팅을 호스트에서 실행하려는 경우 기본값)
- `"all"`: 모든 세션이 샌드박스에서 실행

참고: `"non-main"`은 에이전트 ID가 아닌 `session.mainKey` (기본값 `"main"`)를 기반으로 합니다. 그룹/채널 세션은 자체 키를 사용하므로 비-메인으로 간주되어 샌드박스화됩니다.

## 범위

`agents.defaults.sandbox.scope`는 **몇 개의 컨테이너**가 생성되는지 제어합니다:

- `"session"` (기본값): 세션당 하나의 컨테이너
- `"agent"`: 에이전트당 하나의 컨테이너
- `"shared"`: 모든 샌드박스 세션이 공유하는 하나의 컨테이너

## 워크스페이스 접근

`agents.defaults.sandbox.workspaceAccess`는 **샌드박스가 볼 수 있는 것**을 제어합니다:

- `"none"` (기본값): 도구가 `~/.openclaw/sandboxes` 아래의 샌드박스 워크스페이스를 봅니다
- `"ro"`: 에이전트 워크스페이스를 `/agent`에 읽기 전용으로 마운트
- `"rw"`: 에이전트 워크스페이스를 `/workspace`에 읽기/쓰기로 마운트

## 커스텀 바인드 마운트

`agents.defaults.sandbox.docker.binds`로 추가 호스트 디렉토리를 컨테이너에 마운트합니다.

형식: `host:container:mode` (예: `"/home/user/source:/source:rw"`)

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro"],
        },
      },
    },
  },
}
```

보안 참고사항:

- 바인드는 샌드박스 파일시스템을 우회합니다
- 민감한 마운트 (예: `docker.sock`, 시크릿, SSH 키)는 반드시 `:ro`이어야 합니다

## 이미지 + 설정

기본 이미지: `openclaw-sandbox:bookworm-slim`

한 번 빌드:

```bash
scripts/sandbox-setup.sh
```

기본 이미지에는 Node가 포함되어 있지 **않습니다**. 스킬에 Node가 필요하면 커스텀 이미지를 빌드하거나 `sandbox.docker.setupCommand`를 통해 설치하세요.

기본적으로 샌드박스 컨테이너는 **네트워크 없이** 실행됩니다. `agents.defaults.sandbox.docker.network`로 오버라이드하세요.

## setupCommand (일회성 컨테이너 설정)

`setupCommand`는 샌드박스 컨테이너가 생성된 **후 한 번** 실행됩니다 (매 실행마다가 아님).

일반적인 주의사항:

- 기본 `docker.network`가 `"none"` (송신 없음)이므로 패키지 설치가 실패합니다
- `readOnlyRoot: true`는 쓰기를 방지합니다
- 패키지 설치를 위해 `user`가 root여야 합니다

## 도구 정책 + 탈출구

도구 허용/거부 정책은 샌드박스 규칙 전에 적용됩니다. 도구가 전역적으로 또는 에이전트별로 거부되면 샌드박싱이 이를 되돌리지 않습니다.

`tools.elevated`는 호스트에서 `exec`를 실행하는 명시적 탈출구입니다.

디버깅: `openclaw sandbox explain`으로 유효한 샌드박스 모드, 도구 정책, 수정 설정 키를 검사하세요.

## 최소 활성화 예시

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## 관련 문서

- [샌드박스 설정](/ko-KR/gateway/sandboxing)
- [보안](/ko-KR/gateway/security)
