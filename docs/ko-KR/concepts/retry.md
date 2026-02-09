---
summary: "채널별 HTTP 요청 재시도 정책, 지수 백오프, 지터"
read_when:
  - 메시지 전송 실패 시 재시도 동작을 이해하고 싶을 때
  - 재시도 설정을 변경하고 싶을 때
title: "재시도 정책"
---

# 재시도 정책

채널 메시지 전송이 실패하면 자동 재시도를 수행합니다. 재시도는 HTTP 요청 단위로 적용됩니다.

## 기본값

| 옵션          | 기본값     | 설명                    |
| ------------- | ---------- | ----------------------- |
| `attempts`    | 3          | 최대 시도 횟수          |
| `minDelayMs`  | 채널별     | 최소 대기 시간 (ms)     |
| `maxDelayMs`  | 30000      | 최대 대기 시간 (ms)     |
| `jitter`      | 0.1        | 지터 비율 (10%)         |

지수 백오프로 대기 시간이 증가하며, 지터가 추가되어 동시 재시도를 분산합니다.

## 채널별 기본값

### Telegram

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

재시도 대상: HTTP 429 (속도 제한), 타임아웃, 일시적 오류

### Discord

```json5
{
  channels: {
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

재시도 대상: HTTP 429 (속도 제한)만. 다른 오류는 재시도하지 않습니다.

## 동작 규칙

- 재시도는 **개별 HTTP 요청** 단위로 적용
- 멀티 스텝 플로우 전체를 재시도하지 않음
- 메시지 순서 보장
- 비멱등 작업의 중복 실행 방지

## 다음 단계

- [채널 문제 해결](/ko-KR/channels/troubleshooting) - 채널별 문제 해결
- [채널 개요](/ko-KR/channels) - 지원 채널 전체
