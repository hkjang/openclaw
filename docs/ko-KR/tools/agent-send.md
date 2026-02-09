---
summary: "CLI에서 에이전트를 직접 실행하고 결과를 채널로 전달"
read_when:
  - CLI에서 에이전트를 직접 호출하고 싶을 때
  - 스크립트에서 에이전트를 자동화하고 싶을 때
title: "에이전트 직접 실행 (agent send)"
---

# 에이전트 직접 실행 (agent send)

`openclaw agent` 명령어는 인바운드 채팅 메시지 없이 에이전트를 직접 실행합니다.

## 기본 사용법

```bash
openclaw agent --message "프로젝트 상태를 요약해줘"
```

## 주요 옵션

| 옵션                  | 설명                          |
| --------------------- | ----------------------------- |
| `--message <text>`    | 에이전트에 전달할 메시지      |
| `--to <dest>`         | 대상 지정 (채널:ID)           |
| `--session-id <id>`   | 세션 ID 지정                  |
| `--agent <id>`        | 에이전트 ID 지정              |
| `--local`             | Gateway 없이 로컬 실행        |
| `--deliver`           | 응답을 채널로 전달            |
| `--channel <ch>`      | 전달 채널                     |
| `--thinking <level>`  | 사고 레벨                     |
| `--verbose <mode>`    | 상세 모드                     |
| `--timeout <sec>`     | 타임아웃 (초)                 |
| `--json`              | JSON 형식 출력                |
| `--model <model>`     | 모델 지정                     |

## 실행 예시

### 기본 실행

```bash
openclaw agent --message "안녕하세요"
```

### 특정 에이전트로 실행

```bash
openclaw agent --agent coding --message "테스트 실행해줘"
```

### 채널로 결과 전달

```bash
openclaw agent \
  --message "오늘의 브리핑을 작성해줘" \
  --deliver \
  --channel telegram \
  --to 123456789
```

### 로컬 실행 (Gateway 없이)

```bash
openclaw agent --local --message "간단한 질문"
```

로컬 실행 시 셸의 API 키 환경변수가 필요합니다.

### JSON 출력

```bash
openclaw agent --message "날씨 확인" --json
```

스크립트에서 파싱하기 편한 JSON 형식으로 출력합니다.

### 높은 사고 레벨

```bash
openclaw agent --message "복잡한 분석" --thinking high --model anthropic/claude-opus-4-6
```

## 세션 선택

| 방식                | 설명                          |
| ------------------- | ----------------------------- |
| 미지정              | 기본 에이전트의 기본 세션     |
| `--agent <id>`      | 에이전트 지정                 |
| `--session-id <id>` | 특정 세션 ID                  |
| `--to <dest>`       | 채널:발신자ID 형식            |

## 스크립트 활용

```bash
# 크론에서 매일 아침 브리핑
0 7 * * * openclaw agent --message "오늘의 일정을 요약해줘" --deliver --channel telegram --to 123456789

# CI/CD에서 빌드 결과 알림
openclaw agent --message "빌드 #${BUILD_NUMBER} 완료: ${BUILD_STATUS}" --deliver --channel discord
```

## 다음 단계

- [CLI 레퍼런스](/ko-KR/cli/reference) - 전체 CLI 명령어
- [에이전트 설정](/ko-KR/concepts/agent) - 에이전트 커스터마이징
- [크론 작업](/ko-KR/automation/cron) - 예약 실행
