---
summary: "실행 승인, 허용 목록, 샌드박스 탈출 프롬프트"
read_when:
  - 실행 승인이나 허용 목록을 구성할 때
  - macOS 앱에서 실행 승인 UX를 구현할 때
  - 샌드박스 탈출 프롬프트와 그 의미를 검토할 때
title: "실행 승인"
---

# 실행 승인

실행 승인은 샌드박스화된 에이전트가 실제 호스트(`gateway` 또는 `node`)에서 명령을 실행할 수 있도록 하는 **동반 앱/노드 호스트 가드레일**입니다. 안전 인터록처럼 생각하세요: 정책 + 허용 목록 + (선택적) 사용자 승인이 모두 동의할 때만 명령이 허용됩니다.

실행 승인은 도구 정책 및 elevated 게이팅에 **추가**됩니다 (elevated가 `full`로 설정되면 승인을 건너뜁니다).

동반 앱 UI를 **사용할 수 없는** 경우, 프롬프트가 필요한 모든 요청은 **ask 폴백** (기본값: deny)으로 처리됩니다.

## 적용 위치

실행 승인은 실행 호스트에서 로컬로 적용됩니다:

- **Gateway 호스트** → Gateway 머신의 `openclaw` 프로세스
- **노드 호스트** → 노드 러너 (macOS 동반 앱 또는 헤드리스 노드 호스트)

macOS 분할:

- **노드 호스트 서비스**는 `system.run`을 로컬 IPC를 통해 **macOS 앱**에 전달합니다.
- **macOS 앱**은 승인을 적용하고 UI 컨텍스트에서 명령을 실행합니다.

## 설정 및 저장소

승인은 실행 호스트의 로컬 JSON 파일에 저장됩니다:

`~/.openclaw/exec-approvals.json`

예시 스키마:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO"
        }
      ]
    }
  }
}
```

## 정책 옵션

### 보안 (`exec.security`)

- **deny**: 모든 호스트 실행 요청 차단
- **allowlist**: 허용 목록에 있는 명령만 허용
- **full**: 모든 것 허용 (elevated와 동일)

### 질의 (`exec.ask`)

- **off**: 프롬프트 없음
- **on-miss**: 허용 목록에 일치하지 않을 때만 프롬프트
- **always**: 모든 명령에 대해 프롬프트

### 질의 폴백 (`askFallback`)

프롬프트가 필요하지만 UI에 접근할 수 없을 때:

- **deny**: 차단
- **allowlist**: 허용 목록에 일치하는 경우만 허용
- **full**: 허용

## 허용 목록 (에이전트별)

허용 목록은 **에이전트별**입니다. 여러 에이전트가 있으면 macOS 앱에서 편집할 에이전트를 전환합니다. 패턴은 **대소문자 구분 없는 glob 매치**입니다. 패턴은 **바이너리 경로**로 해석됩니다.

예시:

- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

## 스킬 CLI 자동 허용

**Auto-allow skill CLIs**가 활성화되면, 알려진 스킬에서 참조하는 실행 파일이 노드(macOS 노드 또는 헤드리스 노드 호스트)에서 허용 목록으로 처리됩니다.

## 안전 바이너리 (stdin 전용)

`tools.exec.safeBins`는 허용 목록 모드에서 명시적 허용 목록 항목 **없이** 실행할 수 있는 **stdin 전용** 바이너리의 작은 목록을 정의합니다 (예: `jq`).

기본 안전 바이너리: `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`.

## 승인 흐름

프롬프트가 필요하면 Gateway가 `exec.approval.requested`를 운영자 클라이언트에 브로드캐스트합니다. Control UI와 macOS 앱이 `exec.approval.resolve`로 처리한 다음, Gateway가 승인된 요청을 노드 호스트에 전달합니다.

확인 대화 상자에는 다음이 포함됩니다:

- 명령 + 인자
- 작업 디렉토리
- 에이전트 ID
- 해석된 실행 파일 경로
- 호스트 + 정책 메타데이터

작업:

- **한 번 허용** → 지금 실행
- **항상 허용** → 허용 목록에 추가 + 실행
- **거부** → 차단

## 채팅 채널로 승인 전달

실행 승인 프롬프트를 채팅 채널로 전달하고 `/approve`로 승인할 수 있습니다.

설정:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

채팅에서 답장:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

## 시스템 이벤트

실행 수명 주기는 시스템 메시지로 표시됩니다:

- `Exec running` (명령이 실행 알림 임계값을 초과하는 경우)
- `Exec finished`
- `Exec denied`

## 참고사항

- **full**은 강력합니다. 가능하면 허용 목록을 사용하세요.
- **ask**는 빠른 승인을 허용하면서 사용자를 루프에 유지합니다.
- 에이전트별 허용 목록은 한 에이전트의 승인이 다른 에이전트로 유출되는 것을 방지합니다.

관련:

- [실행 도구](/ko-KR/tools/exec)
- [Elevated 모드](/ko-KR/tools/elevated)
- [스킬](/ko-KR/tools/skills)
