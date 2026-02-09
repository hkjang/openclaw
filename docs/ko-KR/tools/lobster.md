---
summary: "재개 가능한 승인 게이트가 있는 OpenClaw의 타입된 워크플로우 런타임"
read_when:
  - 명시적 승인이 있는 결정론적 다단계 워크플로우가 필요할 때
  - 이전 단계를 다시 실행하지 않고 워크플로우를 재개해야 할 때
title: "Lobster"
---

# Lobster

Lobster는 OpenClaw이 다단계 도구 시퀀스를 명시적 승인 체크포인트가 있는 단일 결정론적 작업으로 실행할 수 있게 하는 워크플로우 셸입니다.

## 왜 Lobster인가

오늘날 복잡한 워크플로우는 많은 왕복 도구 호출이 필요합니다. 각 호출은 토큰을 소비하고 LLM이 모든 단계를 조율해야 합니다. Lobster는 그 조율을 타입된 런타임으로 이동시킵니다:

- **하나의 호출로 대체**: OpenClaw이 하나의 Lobster 도구 호출을 실행하고 구조화된 결과를 받습니다
- **승인 내장**: 부작용(이메일 전송, 댓글 게시)은 명시적으로 승인될 때까지 워크플로우를 중단합니다
- **재개 가능**: 중단된 워크플로우는 토큰을 반환합니다. 승인하고 모든 것을 다시 실행하지 않고 재개합니다

## 작동 방식

OpenClaw이 로컬 `lobster` CLI를 **도구 모드**로 시작하고 stdout에서 JSON 엔벨로프를 파싱합니다. 파이프라인이 승인을 위해 일시 중지되면 도구가 `resumeToken`을 반환하여 나중에 계속할 수 있습니다.

## 패턴: 작은 CLI + JSON 파이프 + 승인

JSON을 말하는 작은 명령을 빌드한 다음 단일 Lobster 호출로 체인합니다:

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

파이프라인이 승인을 요청하면 토큰으로 재개:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

## 워크플로우 파일 (.lobster)

Lobster는 `name`, `args`, `steps`, `env`, `condition`, `approval` 필드가 있는 YAML/JSON 워크플로우 파일을 실행할 수 있습니다:

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

## 설치

Gateway와 **동일한 호스트**에 Lobster CLI를 설치하고 `lobster`가 `PATH`에 있는지 확인하세요.

## 도구 활성화

Lobster는 **선택적** 플러그인 도구입니다 (기본적으로 비활성화).

권장 (추가적, 안전):

```json5
{
  tools: {
    alsoAllow: ["lobster"],
  },
}
```

## 도구 파라미터

### `run`

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

### `resume`

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### 선택적 입력

- `lobsterPath`: Lobster 바이너리의 절대 경로
- `cwd`: 파이프라인의 작업 디렉토리
- `timeoutMs`: 서브프로세스 타임아웃 (기본값: 20000)
- `maxStdoutBytes`: stdout 크기 제한 (기본값: 512000)
- `argsJson`: 워크플로우 파일에 전달할 JSON 문자열

## 출력 엔벨로프

Lobster는 세 가지 상태 중 하나로 JSON 엔벨로프를 반환합니다:

- `ok` → 성공적으로 완료
- `needs_approval` → 일시 중지됨; 재개를 위해 `requiresApproval.resumeToken` 필요
- `cancelled` → 명시적으로 거부 또는 취소됨

## 안전

- **로컬 서브프로세스만** — 플러그인 자체에서 네트워크 호출 없음
- **시크릿 없음** — Lobster는 OAuth를 관리하지 않음
- **샌드박스 인식** — 도구 컨텍스트가 샌드박스화되면 비활성화
- **강화됨** — 타임아웃 및 출력 제한 적용

## 문제 해결

- **`lobster subprocess timed out`** → `timeoutMs`를 늘리거나 파이프라인을 분할하세요
- **`lobster output exceeded maxStdoutBytes`** → `maxStdoutBytes`를 늘리거나 출력 크기를 줄이세요
- **`lobster returned invalid JSON`** → 파이프라인이 도구 모드로 실행되고 JSON만 출력하는지 확인하세요
- **`lobster failed (code …)`** → 터미널에서 동일한 파이프라인을 실행하여 stderr를 검사하세요
