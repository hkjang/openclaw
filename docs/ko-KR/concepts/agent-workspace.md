---
summary: "에이전트 워크스페이스 디렉토리 구조, 부트스트랩 파일, 스킬, 메모리"
read_when:
  - 에이전트의 작업 디렉토리를 이해하고 싶을 때
  - 워크스페이스 파일을 커스터마이징하고 싶을 때
title: "에이전트 워크스페이스"
---

# 에이전트 워크스페이스

워크스페이스는 에이전트의 지침, 스킬, 메모리, 캔버스 파일이 저장되는 디렉토리입니다.

## 기본 위치

```
~/.openclaw/workspace/
```

프로필을 사용하면:

```
~/.openclaw/workspace-<profile>/
```

환경변수: `OPENCLAW_PROFILE=dev` → `~/.openclaw/workspace-dev/`

## 디렉토리 구조

```
~/.openclaw/workspace/
├── AGENTS.md             # 에이전트 동작 지침 (항상 주입)
├── SOUL.md               # 페르소나, 어조, 경계
├── USER.md               # 사용자 정보
├── IDENTITY.md           # 에이전트 이름, 스타일, 이모지
├── TOOLS.md              # 도구 사용 가이드라인
├── HEARTBEAT.md          # 하트비트 점검 지침
├── BOOT.md               # Gateway 재시작 시 실행 (선택)
├── BOOTSTRAP.md          # 최초 실행 시 1회 실행 (선택)
├── MEMORY.md             # 큐레이션된 장기 메모리
├── memory/               # 일별 메모리 로그
│   ├── 2026-02-01.md
│   └── 2026-02-02.md
├── skills/               # 워크스페이스 스킬
│   └── my-skill/
│       └── SKILL.md
└── canvas/               # Canvas UI 파일
```

## 파일 상세

### AGENTS.md (필수)

에이전트에게 항상 주입되는 핵심 지침입니다:

```markdown
# 에이전트 지침

## 역할
당신은 풀스택 개발 어시스턴트입니다.

## 규칙
- 항상 한국어로 응답
- 코드 변경 전 확인 요청
- 민감 정보 노출 금지
```

### SOUL.md

페르소나와 어조를 정의합니다:

```markdown
# 성격
- 친절하고 전문적인 어조
- 기술 설명은 간결하게
- 유머 적절히 사용
```

### USER.md

사용자에 대한 정보를 에이전트에게 제공합니다:

```markdown
# 사용자 정보
- 이름: 홍길동
- 주요 언어: TypeScript, Python
- 에디터: VS Code
```

### IDENTITY.md

에이전트 정체성을 설정합니다:

```markdown
# 정체성
- 이름: 클로
- 이모지: 🦞
- 인사말: "안녕하세요! 무엇을 도와드릴까요?"
```

### TOOLS.md

도구 사용에 대한 가이드라인 (강제가 아닌 안내):

```markdown
# 도구 가이드라인
## bash
- 프로덕션 환경 명령은 확인 후 실행
## browser
- 로그인 필요 사이트 주의
```

### HEARTBEAT.md

하트비트 실행 시 사용되는 점검 지침:

```markdown
# 하트비트 점검
1. 디스크 사용량 90% 이상?
2. 최근 로그에 ERROR?
3. 서비스 실행 중?

정상이면: HEARTBEAT_OK
```

### BOOT.md

Gateway 재시작 시 매번 실행:

```markdown
# 부팅 체크리스트
- 프로젝트 상태 확인
- 의존성 업데이트 필요 여부
```

### BOOTSTRAP.md

최초 실행 시 1회만 실행. 에이전트가 환경을 파악하고 초기 설정을 수행합니다.

크기 제한:

```json5
{
  agents: {
    defaults: {
      bootstrapMaxChars: 20000,  // 기본: 20000
    },
  },
}
```

### memory/ 디렉토리

일별 메모리 로그. 에이전트가 `memory_save` 도구로 작성합니다.

### MEMORY.md

큐레이션된 장기 메모리. 중요한 정보를 수동으로 정리합니다.

## 에이전트별 워크스페이스

```json5
{
  agents: {
    list: [
      {
        id: "main",
        // 기본 워크스페이스 사용
      },
      {
        id: "coding",
        workspace: "~/projects/my-project",  // 커스텀 경로
      },
    ],
  },
}
```

## 워크스페이스에 **없는** 것들

| 항목           | 위치                               |
| -------------- | ---------------------------------- |
| 설정 파일      | `~/.openclaw/openclaw.json`        |
| 인증 정보      | `~/.openclaw/credentials/`         |
| 세션 데이터    | `~/.openclaw/state/<agent>/sessions/` |
| 인증 프로필    | `~/.openclaw/agents/<agent>/agent/`  |

## Git 백업 (권장)

워크스페이스를 Git으로 백업하세요:

```bash
cd ~/.openclaw/workspace
git init
echo "*.log" >> .gitignore
echo "memory/" >> .gitignore
git add -A && git commit -m "Initial workspace"
git remote add origin git@github.com:you/openclaw-workspace.git
git push -u origin main
```

## 다음 단계

- [에이전트 설정](/ko-KR/concepts/agent) - 에이전트 커스터마이징
- [시스템 프롬프트](/ko-KR/concepts/system-prompt) - 프롬프트 조립 과정
- [스킬 만들기](/ko-KR/tools/creating-skills) - 커스텀 스킬 개발
