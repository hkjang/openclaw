---
summary: "Gateway 대시보드 (Control UI) 접근과 인증"
read_when:
  - 대시보드에 접근하고 싶을 때
  - 대시보드 인증을 설정하고 싶을 때
  - unauthorized 오류를 해결하고 싶을 때
title: "대시보드"
---

# 대시보드

Gateway 대시보드(Control UI)는 브라우저에서 채팅, 설정, 세션, 노드를 관리하는 웹 인터페이스입니다.

## 빠른 열기

로컬 Gateway가 실행 중일 때:

```bash
openclaw dashboard
```

이 명령어는:
1. 대시보드 링크를 클립보드에 복사
2. 기본 브라우저에서 열기
3. 헤드리스 서버에서는 SSH 힌트 표시

### 기본 접근 URL

```
http://127.0.0.1:18789/
```

또는:

```
http://localhost:18789/
```

### 커스텀 기본 경로

```json5
{
  gateway: {
    controlUi: {
      basePath: "/admin",  // http://127.0.0.1:18789/admin/
    },
  },
}
```

## 주요 참조 명령어

| 명령어                                        | 설명                          |
| --------------------------------------------- | ----------------------------- |
| `openclaw dashboard`                          | 대시보드 열기                 |
| `openclaw status`                             | Gateway 접근 가능 여부 확인   |
| `openclaw config get gateway.auth.token`      | 현재 토큰 조회                |
| `openclaw doctor --generate-gateway-token`    | 새 토큰 생성                  |

## 인증

대시보드 인증은 **WebSocket 핸드셰이크** 시점에 `connect.params.auth`로 수행됩니다.

### 인증 방식

| 방식      | 설명                           |
| --------- | ------------------------------ |
| **토큰**  | `gateway.auth.token` 값 사용  |
| **비밀번호** | `gateway.auth.password` 사용 |

### 설정

```json5
{
  gateway: {
    auth: {
      token: "your-secure-token",
      // 또는
      password: "your-password",
    },
  },
}
```

### 빠른 경로 (권장)

1. `openclaw doctor --generate-gateway-token` 실행
2. 생성된 토큰이 자동으로 설정에 저장됨
3. 대시보드에서 토큰을 입력하여 연결

### 토큰 기본 동작

| 환경      | 토큰 필요 여부 | 설명                          |
| --------- | -------------- | ----------------------------- |
| **로컬**  | 선택           | localhost에서는 토큰 없이 가능 |
| **원격**  | 필수           | SSH 터널이나 Tailscale 통해   |

## UI 토큰 저장

대시보드는 처음 연결 시 입력한 토큰을 브라우저의 `localStorage`에 저장합니다. 이후 접속 시 자동으로 사용됩니다.

## 보안 권장 사항

Control UI는 **관리자 화면**입니다. 다음 기능에 접근할 수 있습니다:

- 채팅 (에이전트와 대화)
- 설정 변경
- exec 승인 (명령어 실행 허용)
- 세션 관리
- 노드 페어링

### 보안 규칙

| 규칙                         | 권장                              |
| ---------------------------- | --------------------------------- |
| **공개 인터넷에 노출하지 마세요** | ❌ 절대 하지 마세요           |
| **localhost 사용**           | ✅ 가장 안전                      |
| **Tailscale Serve**          | ✅ 편리한 보안 원격 접근          |
| **SSH 터널**                 | ✅ 원격 접근 시 권장              |
| **토큰 설정**                | ✅ 원격 접근 시 필수              |

## 문제 해결

### "unauthorized" / 1008 오류

WebSocket 연결이 거부될 때:

1. **토큰 확인**: `openclaw config get gateway.auth.token`
2. **Gateway 실행 확인**: `openclaw status`
3. **토큰 재생성**: `openclaw doctor --generate-gateway-token`
4. **브라우저 캐시 정리**: localStorage에 저장된 이전 토큰이 유효하지 않을 수 있음

### Gateway에 연결할 수 없음

```bash
# Gateway 상태 확인
openclaw status

# 포트 사용 확인
openclaw status --deep

# 전체 진단
openclaw doctor
```

### 원격에서 접근할 수 없음

1. SSH 터널이 활성인지 확인
2. 토큰이 설정되었는지 확인
3. 방화벽 규칙 확인
4. [원격 접근](/ko-KR/gateway/remote) 가이드 참조

## 다음 단계

- [웹 인터페이스](/ko-KR/web) - Control UI 전체 기능
- [원격 접근](/ko-KR/gateway/remote) - 원격 대시보드 접근
- [보안](/ko-KR/gateway/security) - Gateway 보안 설정
- [Tailscale](/ko-KR/gateway/tailscale) - Tailscale 통합
