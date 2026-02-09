---
summary: "OpenClaw 보안 및 신뢰 개요"
read_when:
  - 보안 아키텍처를 이해하고 싶을 때
  - 취약점 보고 방법을 찾을 때
title: "보안 개요"
---

# 보안 및 신뢰

OpenClaw은 개인 AI 어시스턴트 플랫폼으로, 보안을 핵심 설계 원칙으로 삼고 있습니다.

## 보안 문서

- [실행 승인 흐름](/ko-KR/security/approval-flow) - 명령 실행 승인, 허용 목록, 샌드박스 탈출 프롬프트
- [샌드박싱](/ko-KR/security/sandbox) - Docker 기반 도구 격리
- [위협 모델](/ko-KR/security/threat-model) - MITRE ATLAS 기반 위협 모델

## 보안 계층

OpenClaw은 여러 겹의 보안 경계를 제공합니다:

### 1. 채널 접근 제어

- 디바이스 페어링 (30초 유예 기간)
- `allowFrom` / 허용 목록 검증
- 토큰/비밀번호/Tailscale 인증

### 2. 세션 격리

- 세션 키 = `agent:channel:peer`
- 에이전트별 도구 정책
- 트랜스크립트 로깅

### 3. 도구 실행

- Docker 샌드박스 또는 호스트 (실행 승인)
- 노드 원격 실행
- SSRF 보호 (DNS 고정 + IP 차단)

### 4. 외부 콘텐츠

- 프롬프트 인젝션 완화
- URL 콘텐츠 검증

## 보안 기본값

- DM은 기본적으로 `allowlist` 또는 `pairing` 모드
- 그룹 채팅은 멘션 필요
- 샌드박스는 비-메인 세션에 대해 활성화 가능
- 실행 승인은 기본적으로 `deny`

## 취약점 보고

보안 취약점을 발견하셨다면:

- [trust.openclaw.ai](https://trust.openclaw.ai)에서 전체 보고 지침을 확인하세요
- Discord #security 채널에서 문의하세요

## 관련 문서

- [Gateway 보안](/ko-KR/gateway/security)
- [샌드박싱](/ko-KR/gateway/sandboxing)
- [인증](/ko-KR/gateway/authentication)
