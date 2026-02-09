---
summary: "채팅의 세션 관리 규칙, 키, 지속성"
read_when:
  - 세션 처리나 저장소를 수정할 때
title: "세션 관리"
---

# 세션 관리

OpenClaw은 **에이전트당 하나의 직접 채팅 세션**을 기본으로 취급합니다. 직접 채팅은 `agent:<agentId>:<mainKey>` (기본값 `main`)로 축소되며, 그룹/채널 채팅은 자체 키를 갖습니다.

`session.dmScope`로 **직접 메시지**가 그룹화되는 방식을 제어합니다:

- `main` (기본값): 모든 DM이 연속성을 위해 메인 세션을 공유
- `per-peer`: 채널 간 발신자 ID로 격리
- `per-channel-peer`: 채널 + 발신자로 격리 (멀티 유저 인박스에 권장)
- `per-account-channel-peer`: 계정 + 채널 + 발신자로 격리 (멀티 계정 인박스에 권장)

`session.identityLinks`를 사용하여 프로바이더 접두사가 있는 피어 ID를 정규 ID에 매핑하면, 동일한 사람이 여러 채널에서 DM 세션을 공유합니다.

## 안전한 DM 모드 (멀티 유저 설정에 권장)

> **보안 경고:** 에이전트가 **여러 사람**으로부터 DM을 받을 수 있는 경우, 안전한 DM 모드를 활성화하는 것을 강력히 권장합니다. 활성화하지 않으면 모든 사용자가 동일한 대화 컨텍스트를 공유하여 사용자 간에 개인 정보가 유출될 수 있습니다.

**수정 방법:** `dmScope`를 설정하여 사용자별로 세션을 격리:

```json5
{
  session: {
    dmScope: "per-channel-peer",
  },
}
```

**활성화해야 하는 경우:**

- 여러 발신자에 대한 페어링 승인이 있을 때
- 여러 항목이 있는 DM 허용 목록을 사용할 때
- `dmPolicy: "open"`을 설정한 경우
- 여러 전화번호나 계정이 에이전트에 메시지를 보낼 수 있을 때

## Gateway가 진실의 원천

모든 세션 상태는 **Gateway가 소유**합니다. UI 클라이언트(macOS 앱, WebChat 등)는 로컬 파일을 읽는 대신 Gateway에서 세션 목록과 토큰 수를 쿼리해야 합니다.

## 상태 저장 위치

- **Gateway 호스트**:
  - 저장소 파일: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (에이전트별)
  - 트랜스크립트: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`
- 저장소는 `sessionKey -> { sessionId, updatedAt, ... }` 맵입니다. 항목 삭제는 안전합니다.

## 세션 프루닝

OpenClaw은 기본적으로 LLM 호출 직전에 인메모리 컨텍스트에서 **이전 도구 결과**를 트리밍합니다. JSONL 기록은 재작성하지 **않습니다**.

## 세션 재설정

```json5
{
  session: {
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetTriggers: ["/new", "/reset"],
  },
}
```

## 관련 문서

- [세션 도구](/ko-KR/concepts/session-tool)
- [설정](/ko-KR/gateway/configuration)
