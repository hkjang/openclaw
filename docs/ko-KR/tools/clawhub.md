---
summary: "ClawHub 가이드: 공개 스킬 레지스트리 + CLI 워크플로우"
read_when:
  - 새로운 사용자에게 ClawHub을 소개할 때
  - 스킬을 검색, 설치, 게시할 때
  - ClawHub CLI 플래그와 동기화 동작을 설명할 때
title: "ClawHub"
---

# ClawHub

ClawHub은 **OpenClaw의 공개 스킬 레지스트리**입니다. 무료 서비스이며, 모든 스킬은 공개적이고 누구나 공유하고 재사용할 수 있습니다. 스킬은 `SKILL.md` 파일(+ 지원 텍스트 파일)이 있는 폴더입니다. 웹 앱에서 스킬을 둘러보거나 CLI를 사용하여 검색, 설치, 업데이트, 게시할 수 있습니다.

사이트: [clawhub.ai](https://clawhub.ai)

## ClawHub이란

- OpenClaw 스킬을 위한 공개 레지스트리
- 스킬 번들과 메타데이터의 버전 관리 저장소
- 검색, 태그, 사용 신호를 위한 발견 표면

## 누구를 위한 것인가

OpenClaw 에이전트에 새로운 기능을 추가하고 싶다면, ClawHub이 스킬을 찾고 설치하는 가장 쉬운 방법입니다:

- 평문으로 스킬 검색
- 워크스페이스에 스킬 설치
- 나중에 하나의 명령으로 스킬 업데이트
- 자신의 스킬을 게시하여 백업

## 빠른 시작

1. CLI 설치:

```bash
npm i -g clawhub
```

2. 필요한 것 검색:

```bash
clawhub search "calendar"
```

3. 스킬 설치:

```bash
clawhub install <skill-slug>
```

4. 새 스킬을 인식하도록 새 OpenClaw 세션을 시작합니다.

## OpenClaw과의 통합

기본적으로 CLI는 현재 작업 디렉토리의 `./skills`에 스킬을 설치합니다. OpenClaw 워크스페이스가 구성되어 있으면 `clawhub`은 `--workdir`을 오버라이드하지 않는 한 해당 워크스페이스로 폴백합니다. OpenClaw은 `<workspace>/skills`에서 워크스페이스 스킬을 로드합니다.

## CLI 명령

### 인증

```bash
clawhub login          # 브라우저 흐름
clawhub login --token <token>  # 토큰으로 로그인
clawhub logout
clawhub whoami
```

### 검색

```bash
clawhub search "query"
clawhub search "query" --limit 10
```

### 설치

```bash
clawhub install <slug>
clawhub install <slug> --version <version>
clawhub install <slug> --force
```

### 업데이트

```bash
clawhub update <slug>
clawhub update --all
```

### 목록

```bash
clawhub list
```

### 게시

```bash
clawhub publish <path> --slug <slug> --name "My Skill" --version 1.0.0
```

### 동기화

```bash
clawhub sync              # 로컬 스킬 스캔 + 새로운/업데이트된 것 게시
clawhub sync --all        # 프롬프트 없이 모두 업로드
clawhub sync --dry-run    # 업로드될 것만 표시
```

## 보안 및 모더레이션

ClawHub은 기본적으로 개방되어 있습니다. 누구나 스킬을 업로드할 수 있지만, 게시하려면 GitHub 계정이 최소 1주일 이상 되어야 합니다.

보고 및 모더레이션:

- 로그인한 사용자는 누구나 스킬을 보고할 수 있습니다
- 보고 이유가 필요하며 기록됩니다
- 3개 이상의 고유 보고가 있는 스킬은 기본적으로 자동 숨김됩니다
- 모더레이터는 스킬을 숨기기/숨김 해제/삭제하거나 사용자를 차단할 수 있습니다

## 환경변수

- `CLAWHUB_SITE`: 사이트 URL 오버라이드
- `CLAWHUB_REGISTRY`: 레지스트리 API URL 오버라이드
- `CLAWHUB_CONFIG_PATH`: CLI가 토큰/설정을 저장하는 위치 오버라이드
- `CLAWHUB_WORKDIR`: 기본 작업 디렉토리 오버라이드
- `CLAWHUB_DISABLE_TELEMETRY=1`: `sync`에서 텔레메트리 비활성화
