---
summary: "세션 목록 조회, 기록 가져오기, 교차 세션 메시지 전송을 위한 에이전트 세션 도구"
read_when:
  - 세션 도구를 추가하거나 수정할 때
title: "세션 도구"
---

# 세션 도구

에이전트가 세션을 나열하고, 기록을 가져오고, 다른 세션에 메시지를 보낼 수 있는 작고 오용하기 어려운 도구 세트입니다.

## 도구 이름

- `sessions_list` — 세션 목록 조회
- `sessions_history` — 세션 기록 조회
- `sessions_send` — 다른 세션에 메시지 전송
- `sessions_spawn` — 새 세션 생성

## 키 모델

- 메인 직접 채팅: 리터럴 키 `"main"`
- 그룹 채팅: `agent:<agentId>:<channel>:group:<id>`
- Cron 작업: `cron:<job.id>`
- 훅: `hook:<uuid>`
- 노드 세션: `node-<nodeId>`

## sessions_list

세션을 행 배열로 나열합니다.

파라미터:

- `kinds?: string[]` — 필터: `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` — 최대 행 수
- `activeMinutes?: number` — N분 이내에 업데이트된 세션만
- `messageLimit?: number` — 0 = 메시지 없음 (기본값); >0 = 마지막 N개 메시지 포함

행 형태:

- `key`, `kind`, `channel`, `displayName`, `updatedAt`, `sessionId`
- `model`, `contextTokens`, `totalTokens`

## sessions_history

하나의 세션에 대한 트랜스크립트를 가져옵니다.

파라미터:

- `sessionKey` (필수)
- `limit?: number` — 최대 메시지 수
- `includeTools?: boolean` (기본값 false)

## sessions_send

다른 세션에 메시지를 보냅니다.

파라미터:

- `sessionKey` (필수)
- `text` (필수)
- `deliver?: boolean` — true이면 연결된 채널에도 전달

## sessions_spawn

새 세션을 생성합니다.

파라미터:

- `prompt` (필수)
- `model?: string`
- `sessionKey?: string`

## 샌드박스 가시성

샌드박스화된 에이전트 세션에서 실행 시, 세션 도구는 기본적으로 생성된 세션만 표시됩니다.

## 관련 문서

- [세션 관리](/ko-KR/concepts/session)
