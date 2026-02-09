---
summary: "Chrome 확장 프로그램: OpenClaw이 기존 Chrome 탭을 제어하게 하기"
read_when:
  - 에이전트가 기존 Chrome 탭을 제어하게 하고 싶을 때
  - 원격 Gateway + 로컬 브라우저 자동화가 필요할 때
  - 브라우저 제어의 보안 영향을 이해하고 싶을 때
title: "Chrome 확장 프로그램"
---

# Chrome 확장 프로그램 (브라우저 릴레이)

OpenClaw Chrome 확장 프로그램은 에이전트가 별도의 openclaw 관리 Chrome 프로필 대신 **기존 Chrome 탭**(일반 Chrome 창)을 제어할 수 있게 합니다.

연결/분리는 **단일 Chrome 툴바 버튼**으로 이루어집니다.

## 구성 요소

세 부분으로 구성됩니다:

- **브라우저 제어 서비스** (Gateway 또는 노드): 에이전트/도구가 호출하는 API
- **로컬 릴레이 서버** (루프백 CDP): 제어 서버와 확장 프로그램 사이의 브릿지 (기본 `http://127.0.0.1:18792`)
- **Chrome MV3 확장 프로그램**: `chrome.debugger`로 활성 탭에 연결하고 CDP 메시지를 릴레이에 파이프

## 설치 (언팩)

1. 확장 프로그램을 안정적인 로컬 경로에 설치:

```bash
openclaw browser extension install
```

2. 설치된 확장 프로그램 디렉토리 경로 출력:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`
   - "개발자 모드" 활성화
   - "압축해제된 확장 프로그램을 로드합니다" → 위에서 출력된 디렉토리 선택

4. 확장 프로그램을 고정합니다.

## 업데이트

OpenClaw 업그레이드 후:

- `openclaw browser extension install`을 다시 실행하여 파일 새로 고침
- Chrome → `chrome://extensions` → 확장 프로그램에서 "새로 고침" 클릭

## 사용 (추가 설정 불필요)

OpenClaw은 기본 포트의 확장 프로그램 릴레이를 대상으로 하는 `chrome`이라는 내장 브라우저 프로필과 함께 제공됩니다.

사용:

- CLI: `openclaw browser --browser-profile chrome tabs`
- 에이전트 도구: `browser`에 `profile="chrome"`

## 연결 / 분리 (툴바 버튼)

- OpenClaw이 제어하길 원하는 탭을 엽니다.
- 확장 프로그램 아이콘을 클릭합니다.
  - 뱃지가 `ON`이면 연결됨.
- 다시 클릭하면 분리됩니다.

## 어떤 탭을 제어하나요?

- "보고 있는 탭"을 자동으로 제어하지 **않습니다**.
- 툴바 버튼을 클릭하여 **명시적으로 연결한 탭만** 제어합니다.
- 전환하려면: 다른 탭을 열고 거기서 확장 프로그램 아이콘을 클릭합니다.

## 뱃지 + 일반적인 오류

- `ON`: 연결됨; OpenClaw이 해당 탭을 제어할 수 있음
- `…`: 로컬 릴레이에 연결 중
- `!`: 릴레이에 접근할 수 없음 (가장 일반적: 이 머신에서 브라우저 릴레이 서버가 실행되지 않음)

`!`가 표시되면:

- Gateway가 로컬에서 실행 중인지 확인하거나
- Gateway가 다른 곳에서 실행되면 이 머신에서 노드 호스트를 실행하세요

## 원격 Gateway

### 로컬 Gateway (Chrome과 동일 머신) — 보통 **추가 단계 없음**

Gateway가 Chrome과 동일한 머신에서 실행되면 루프백에서 브라우저 제어 서비스를 시작하고 릴레이 서버를 자동 시작합니다.

### 원격 Gateway (Gateway가 다른 곳에서 실행) — **노드 호스트 실행**

Gateway가 다른 머신에서 실행되면 Chrome을 실행하는 머신에서 노드 호스트를 시작합니다.

## 샌드박싱 (도구 컨테이너)

에이전트 세션이 샌드박스화된 경우 `browser` 도구가 제한될 수 있습니다:

- 기본적으로 샌드박스 세션은 호스트 Chrome이 아닌 **샌드박스 브라우저**를 대상으로 합니다
- Chrome 확장 프로그램 릴레이는 **호스트** 브라우저 제어 서버 제어가 필요합니다

호스트 브라우저 제어 허용:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

## 보안 영향 (반드시 읽으세요)

이것은 강력하고 위험합니다. 모델에 "브라우저에 손을 주는 것"처럼 취급하세요.

- 확장 프로그램은 Chrome의 디버거 API(`chrome.debugger`)를 사용합니다. 연결되면 모델이:
  - 해당 탭에서 클릭/입력/탐색
  - 페이지 콘텐츠 읽기
  - 탭의 로그인 세션이 접근할 수 있는 모든 것에 접근

권장 사항:

- 확장 프로그램 릴레이 사용 시 전용 Chrome 프로필 사용 (개인 브라우징과 분리)
- Gateway와 노드 호스트를 tailnet 전용으로 유지
- LAN(`0.0.0.0`) 및 공개 Funnel을 통한 릴레이 포트 노출 방지

관련:

- [브라우저 도구](/ko-KR/tools/browser)
- [보안](/ko-KR/gateway/security)
- [Tailscale](/ko-KR/gateway/tailscale)
