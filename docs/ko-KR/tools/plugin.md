---
summary: "OpenClaw 플러그인/확장: 발견, 설정, 안전"
read_when:
  - 플러그인/확장을 추가하거나 수정할 때
  - 플러그인 설치 또는 로드 규칙을 문서화할 때
title: "플러그인"
---

# 플러그인 (확장)

## 빠른 시작

플러그인은 OpenClaw을 추가 기능(명령, 도구, Gateway RPC)으로 확장하는 **작은 코드 모듈**입니다.

대부분의 경우 핵심 OpenClaw에 아직 내장되지 않은 기능이 필요하거나 선택적 기능을 메인 설치에서 분리하고 싶을 때 플러그인을 사용합니다.

빠른 경로:

1. 이미 로드된 것을 확인:

```bash
openclaw plugins list
```

2. 공식 플러그인 설치 (예: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

3. Gateway를 재시작한 후 `plugins.entries.<id>.config`에서 설정합니다.

## 공식 플러그인

- **Microsoft Teams** — `@openclaw/msteams`
- **Memory (Core)** — 내장 메모리 검색 플러그인
- **Memory (LanceDB)** — 내장 장기 메모리 플러그인
- **Voice Call** — `@openclaw/voice-call`
- **Matrix** — `@openclaw/matrix`
- **Zalo Personal** — `@openclaw/zalouser`
- **Google Antigravity OAuth** — 내장 (기본 비활성화)
- **Qwen OAuth** — 내장 (기본 비활성화)
- **Copilot Proxy** — 내장 (기본 비활성화)

플러그인은 다음을 등록할 수 있습니다:

- Gateway RPC 메서드
- Gateway HTTP 핸들러
- 에이전트 도구
- CLI 명령
- 백그라운드 서비스
- 설정 검증
- **스킬** (플러그인 매니페스트에 `skills` 디렉토리 나열)
- **자동 응답 명령** (AI 에이전트 호출 없이 실행)

플러그인은 Gateway와 **동일 프로세스**에서 실행되므로 신뢰할 수 있는 코드로 취급하세요.

## 발견 및 우선순위

OpenClaw은 다음 순서로 스캔합니다:

1. **설정 경로**: `plugins.load.paths`
2. **워크스페이스 확장**: `<workspace>/.openclaw/extensions/`
3. **전역 확장**: `~/.openclaw/extensions/`
4. **내장 확장**: OpenClaw과 함께 제공 (**기본 비활성화**)

내장 플러그인은 `plugins.entries.<id>.enabled` 또는 `openclaw plugins enable <id>`로 명시적으로 활성화해야 합니다.

## 설정

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

필드:

- `enabled`: 마스터 토글 (기본값: true)
- `allow`: 허용 목록 (선택적)
- `deny`: 거부 목록 (선택적, deny가 우선)
- `load.paths`: 추가 플러그인 파일/디렉토리
- `entries.<id>`: 플러그인별 토글 + 설정

설정 변경은 **Gateway 재시작이 필요합니다**.

## 플러그인 슬롯 (독점 카테고리)

일부 플러그인 카테고리는 **독점적**입니다 (한 번에 하나만 활성). `plugins.slots`로 슬롯을 소유할 플러그인을 선택합니다:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // 또는 "none"으로 메모리 플러그인 비활성화
    },
  },
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install @openclaw/voice-call  # npm에서 설치
openclaw plugins install ./extensions/voice-call  # 로컬 경로
openclaw plugins install -l ./extensions/voice-call  # 개발용 링크
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

## 배포 (npm)

- 플러그인: `@openclaw/*` 아래 별도 npm 패키지
- `openclaw plugins install <npm-spec>`로 설치
- 플러그인 `package.json`에 `openclaw.extensions`가 포함되어야 함

## 안전 참고사항

플러그인은 Gateway와 동일 프로세스에서 실행됩니다. 신뢰할 수 있는 코드로 취급하세요:

- 신뢰하는 플러그인만 설치하세요
- `plugins.allow` 허용 목록을 사용하세요
- 변경 후 Gateway를 재시작하세요
