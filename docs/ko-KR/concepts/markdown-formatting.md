---
summary: "아웃바운드 채널을 위한 Markdown 포맷팅 파이프라인"
read_when:
  - 아웃바운드 채널의 Markdown 포맷팅이나 청킹을 변경할 때
  - 새 채널 포맷터나 스타일 매핑을 추가할 때
title: "Markdown 포맷팅"
---

# Markdown 포맷팅

OpenClaw은 아웃바운드 Markdown을 공유 중간 표현(IR)으로 변환한 다음 채널별 출력을 렌더링합니다. IR은 원본 텍스트를 유지하면서 스타일/링크 스팬을 전달하여 청킹과 렌더링이 채널 간에 일관되게 유지됩니다.

## 목표

- **일관성:** 한 번의 파싱, 여러 렌더러
- **안전한 청킹:** 렌더링 전에 텍스트를 분할하여 인라인 포맷팅이 청크 간에 깨지지 않도록
- **채널 적합:** 동일한 IR을 Slack mrkdwn, Telegram HTML, Signal 스타일 범위로 매핑

## 파이프라인

1. **Markdown → IR 파싱**
   - IR은 일반 텍스트 + 스타일 스팬(볼드/이탤릭/취소선/코드/스포일러) + 링크 스팬
   - 오프셋은 UTF-16 코드 단위 (Signal 스타일 범위와 정렬)

2. **IR 청킹 (포맷 우선)**
   - 청킹은 렌더링 전에 IR 텍스트에서 발생
   - 인라인 포맷팅이 청크 간에 분할되지 않음

3. **채널별 렌더링**
   - **Slack:** mrkdwn 토큰, 링크는 `<url|label>`
   - **Telegram:** HTML 태그 (`<b>`, `<i>`, `<s>`, `<code>`, `<a href>`)
   - **Signal:** 일반 텍스트 + `text-style` 범위

## IR 예시

입력 Markdown:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (개략):

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## 사용 위치

- Slack, Telegram, Signal 아웃바운드 어댑터가 IR에서 렌더링
- 다른 채널(WhatsApp, iMessage, MS Teams, Discord)은 일반 텍스트나 자체 포맷팅 규칙 사용

## 테이블 처리

Markdown 테이블은 채팅 클라이언트 간에 일관되게 지원되지 않습니다. `markdown.tables`로 채널별 변환을 제어합니다:

- `code`: 테이블을 코드 블록으로 렌더링 (대부분의 채널 기본값)
- `bullets`: 각 행을 불릿 포인트로 변환 (Signal + WhatsApp 기본값)
- `off`: 테이블 파싱 및 변환 비활성화

설정:

```json5
{
  channels: {
    discord: {
      markdown: { tables: "code" },
    },
    whatsapp: {
      markdown: { tables: "bullets" },
    },
  },
}
```

## 관련 문서

- [설정 가이드](/ko-KR/gateway/configuration)
