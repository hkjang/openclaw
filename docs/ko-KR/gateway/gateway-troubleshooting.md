---
summary: "Gateway, 채널, 자동화, 노드, 브라우저에 대한 심층 문제 해결 런북"
read_when:
  - 문제 해결 허브에서 심층 진단을 위해 여기로 안내받았을 때
  - 정확한 명령어가 포함된 안정적인 증상 기반 런북이 필요할 때
title: "문제 해결"
---

# Gateway 문제 해결

이 페이지는 심층 런북입니다. 빠른 분류 흐름을 원하면 [도움말/FAQ](/ko-KR/help/faq)에서 시작하세요.

## 진단 명령 순서

먼저 이 명령들을 순서대로 실행하세요:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

정상 신호:

- `openclaw gateway status`에서 `Runtime: running` 및 `RPC probe: ok` 표시
- `openclaw doctor`에서 차단하는 설정/서비스 문제 없음 보고
- `openclaw channels status --probe`에서 연결됨/준비됨 채널 표시

## 응답 없음

채널은 작동하지만 아무 응답이 없으면, 재연결하기 전에 라우팅과 정책을 확인하세요.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list <channel>
openclaw config get channels
openclaw logs --follow
```

확인 사항:

- DM 발신자에 대한 페어링 대기 중
- 그룹 멘션 게이팅 (`requireMention`, `mentionPatterns`)
- 채널/그룹 허용 목록 불일치

일반적인 로그 서명:

- `drop guild message (mention required` → 멘션될 때까지 그룹 메시지 무시
- `pairing request` → 발신자 승인 필요
- `blocked` / `allowlist` → 정책에 의해 발신자/채널이 필터됨

## 대시보드 Control UI 연결

대시보드/Control UI가 연결되지 않을 때 URL, 인증 모드, 보안 컨텍스트를 검증하세요.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

확인 사항:

- 올바른 프로브 URL과 대시보드 URL
- 클라이언트와 Gateway 간 인증 모드/토큰 불일치
- 디바이스 아이덴티티가 필요한 곳에서 HTTP 사용

일반적인 로그 서명:

- `device identity required` → 비보안 컨텍스트 또는 디바이스 인증 누락
- `unauthorized` / 재연결 루프 → 토큰/비밀번호 불일치
- `gateway connect failed:` → 잘못된 호스트/포트/URL 대상

## Gateway 서비스 미실행

서비스가 설치되었지만 프로세스가 유지되지 않을 때 사용합니다.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

확인 사항:

- 종료 힌트가 있는 `Runtime: stopped`
- 서비스 설정 불일치 (`Config (cli)` vs `Config (service)`)
- 포트/리스너 충돌

일반적인 로그 서명:

- `Gateway start blocked: set gateway.mode=local` → 로컬 Gateway 모드 미활성화
- `refusing to bind gateway ... without auth` → 인증 없이 비-루프백 바인드
- `another gateway instance is already listening` / `EADDRINUSE` → 포트 충돌

## 채널 연결됨 - 메시지 미전달

채널 상태는 연결됨이지만 메시지 흐름이 멈췄을 때, 정책, 권한, 채널별 전달 규칙에 집중하세요.

```bash
openclaw channels status --probe
openclaw pairing list <channel>
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

확인 사항:

- DM 정책 (`pairing`, `allowlist`, `open`, `disabled`)
- 그룹 허용 목록 및 멘션 요구사항
- 누락된 채널 API 권한/스코프

일반적인 로그 서명:

- `mention required` → 그룹 멘션 정책에 의해 메시지 무시
- `pairing` / 승인 대기 → 발신자 미승인
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → 채널 인증/권한 문제

## Cron 및 하트비트 전달

Cron이나 하트비트가 실행되지 않거나 전달되지 않으면, 먼저 스케줄러 상태를 확인한 후 전달 대상을 확인하세요.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

확인 사항:

- Cron 활성화 및 다음 실행 시간 존재
- 작업 실행 기록 상태 (`ok`, `skipped`, `error`)
- 하트비트 건너뛰기 이유 (`quiet-hours`, `requests-in-flight`, `alerts-disabled`)

## 노드 페어링 후 도구 실패

노드가 페어링되었지만 도구가 실패하면, 포그라운드, 권한, 승인 상태를 확인하세요.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

일반적인 로그 서명:

- `NODE_BACKGROUND_UNAVAILABLE` → 노드 앱이 포그라운드에 있어야 함
- `*_PERMISSION_REQUIRED` → OS 권한 누락
- `SYSTEM_RUN_DENIED: approval required` → 실행 승인 대기
- `SYSTEM_RUN_DENIED: allowlist miss` → 허용 목록에 의해 명령 차단

## 브라우저 도구 실패

Gateway 자체는 정상이지만 브라우저 도구 작업이 실패할 때 사용합니다.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

일반적인 로그 서명:

- `Failed to start Chrome CDP on port` → 브라우저 프로세스 시작 실패
- `browser.executablePath not found` → 구성된 경로가 유효하지 않음
- `Chrome extension relay is running, but no tab is connected` → 확장 프로그램 릴레이 미연결
- `Browser attachOnly is enabled ... not reachable` → attach-only 프로필에 접근 가능한 대상 없음

## 업그레이드 후 문제 발생

대부분의 업그레이드 후 문제는 설정 드리프트 또는 이제 적용되는 더 엄격한 기본값입니다.

### 1) 인증 및 URL 오버라이드 동작 변경

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

확인 사항:

- `gateway.mode=remote`이면 CLI 호출이 로컬 서비스 대신 원격을 대상으로 할 수 있음
- 명시적 `--url` 호출은 저장된 자격 증명으로 폴백하지 않음

### 2) 바인드 및 인증 가드레일 강화

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

확인 사항:

- 비-루프백 바인드(`lan`, `tailnet`, `custom`)는 인증 구성 필요
- 이전 키 `gateway.token`은 `gateway.auth.token`을 대체하지 않음

### 3) 페어링 및 디바이스 아이덴티티 상태 변경

```bash
openclaw devices list
openclaw pairing list <channel>
openclaw logs --follow
openclaw doctor
```

서비스 설정과 런타임이 확인 후에도 일치하지 않으면, 동일한 프로필/상태 디렉토리에서 서비스 메타데이터를 재설치하세요:

```bash
openclaw gateway install --force
openclaw gateway restart
```
