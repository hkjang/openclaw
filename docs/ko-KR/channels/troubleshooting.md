---
summary: "채널별 일반적인 문제와 해결 방법"
read_when:
  - 채널 연결이나 메시지 전송에 문제가 있을 때
title: "채널 문제 해결"
---

# 채널 문제 해결

각 채널별 일반적인 문제와 해결 방법입니다.

## 공통 진단

```bash
# 모든 채널 상태
openclaw channels status

# 특정 채널 상태
openclaw channels status whatsapp

# 채널 재연결
openclaw channels login

# Doctor 진단
openclaw doctor
```

## WhatsApp

### QR 코드 스캔 후 연결 안 됨

1. WhatsApp이 최신 버전인지 확인
2. 기존 연결된 기기 목록에서 OpenClaw 제거 후 재시도
3. `openclaw channels login whatsapp`로 새 QR 생성

### 연결이 자주 끊김

1. 네트워크 안정성 확인
2. WhatsApp Web 세션 제한 확인 (기기당 4개)
3. Gateway 로그 확인: `openclaw logs --filter whatsapp`

### 그룹 메시지 무시됨

1. `requireMention` 설정 확인
2. 멘션 패턴 확인: `messages.groupChat.mentionPatterns`
3. 그룹 정책 확인: `groupPolicy`, `groupAllowlist`

### 미디어 전송 실패

1. 파일 크기 제한 확인 (WhatsApp: 이미지 16MB, 비디오 64MB)
2. 지원되는 형식인지 확인
3. 네트워크 연결 확인

## Telegram

### 봇이 응답하지 않음

1. 봇 토큰 유효성 확인: `openclaw channels status telegram`
2. BotFather에서 봇이 비활성화되지 않았는지 확인
3. 폴링/웹훅 모드 확인

### 웹훅 실패

1. 웹훅 URL이 공개적으로 접근 가능한지 확인
2. HTTPS 인증서 유효성 확인
3. 폴링 모드로 전환 시도

### 봇 프라이버시 모드

그룹에서 모든 메시지를 수신하지 못하면:
1. BotFather → `/mybots` → Bot Settings → Group Privacy → **Turn off**
2. 봇을 그룹에서 제거 후 다시 추가

### 속도 제한

Telegram API 속도 제한에 걸리면:
1. 메시지 전송 빈도 줄이기
2. 재시도 설정 조정: `channels.telegram.retry`

## Discord

### 봇이 서버에 안 보임

1. 올바른 인텐트가 활성화되었는지 확인:
   - `GUILDS`, `GUILD_MESSAGES`, `MESSAGE_CONTENT`
2. Developer Portal → Bot → Privileged Gateway Intents 활성화
3. 올바른 권한으로 봇 초대 URL 재생성

### 메시지를 읽지 못함

Discord에서 `MESSAGE_CONTENT` 인텐트가 필요합니다:
1. Discord Developer Portal 접속
2. Bot → Privileged Gateway Intents → **Message Content Intent** 활성화

### 속도 제한 (429)

1. 메시지 전송 빈도 확인
2. Discord 재시도 설정: `channels.discord.retry`
3. 스트리밍 편집 간격 늘리기

## Slack

### OAuth 연결 실패

1. 앱 자격 증명 (Client ID, Client Secret) 확인
2. 리다이렉트 URL 확인
3. 필요한 스코프 확인

### Socket Mode 연결 끊김

1. 앱 토큰 유효성 확인
2. Socket Mode가 활성화되었는지 확인
3. Gateway 로그 확인: `openclaw logs --filter slack`

### 봇이 멘션에 반응하지 않음

1. 이벤트 구독 확인: `app_mention`, `message.im`
2. 봇 스코프 확인
3. 채널에 봇이 추가되었는지 확인

## Signal

### 연결 실패

1. signal-cli가 설치되었는지 확인
2. 전화번호 등록 상태 확인
3. signal-cli 버전 호환성 확인

## iMessage

### macOS에서 작동하지 않음

1. macOS에서만 지원됩니다
2. 터미널/앱에 전체 디스크 접근 권한 부여
3. iMessage가 로그인되었는지 확인

### BlueBubbles 대안

macOS 없이 iMessage를 사용하려면 [BlueBubbles](https://bluebubbles.app/) 확장을 사용하세요.

## 일반 문제

### 모든 채널이 연결되지 않음

```bash
# Gateway 실행 확인
openclaw status --deep

# Doctor 진단
openclaw doctor --repair
```

### 메시지가 전송되지 않음

1. `allowFrom` 설정 확인 (발신자 허용 목록)
2. 그룹 정책 확인
3. 에이전트 인증 프로필 확인: `openclaw models auth list`

### 느린 응답

1. 모델 응답 시간 확인
2. 컨텍스트 크기 확인: `/usage tokens`
3. 네트워크 지연 확인
4. `/compact`로 컨텍스트 축소

## 다음 단계

- [문제 해결](/ko-KR/help/troubleshooting) - 일반적인 문제 해결
- [헬스 체크](/ko-KR/gateway/health) - 상태 진단
- [로깅](/ko-KR/gateway/logging) - 로그 확인
