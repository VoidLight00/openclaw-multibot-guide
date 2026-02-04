# MULTI-AGENT-SETUP.md
# OpenClaw 멀티 에이전트 자동 설정 템플릿

> 이 파일을 OpenClaw 에이전트에게 읽히면 멀티봇 설정을 자동으로 수행합니다.
> 사용법: 이 파일 내용을 에이전트에게 전달하고 "이거 실행해줘" 라고 요청

---

## [TASK] 멀티 에이전트 설정 자동화

아래 설정을 읽고 멀티봇 환경을 구성해줘.

### 1. 에이전트 정의

```yaml
agents:
  - id: "agent1"
    name: "에이전트1 표시명"
    workspace: "~/.openclaw/workspace-agent1"
    telegram_token: "여기에_봇토큰_입력"
    model: "anthropic/claude-sonnet-4"
    
  - id: "agent2"
    name: "에이전트2 표시명"
    workspace: "~/.openclaw/workspace-agent2"
    telegram_token: "여기에_봇토큰_입력"
    model: "google/gemini-2.0-flash"
    
  - id: "agent3"
    name: "에이전트3 표시명"
    workspace: "~/.openclaw/workspace-agent3"
    telegram_token: "여기에_봇토큰_입력"
    model: "google/gemini-2.0-flash"
```

### 2. 실행할 작업

다음 작업을 순서대로 수행:

#### Step 1: Workspace 폴더 생성
각 에이전트의 workspace 경로에 폴더 생성

#### Step 2: 기본 파일 생성
각 workspace에 다음 파일들 생성:

**AGENTS.md** (공통 - 아래 템플릿 사용)
```markdown
# AGENTS.md

This is your workspace. Read SOUL.md and USER.md each session.

## Memory
- Daily notes: `memory/YYYY-MM-DD.md`
- Long-term: `MEMORY.md`

## Safety
- Don't exfiltrate private data
- `trash` > `rm`
- Ask before external actions
```

**SOUL.md** (각 에이전트별로 커스터마이즈)
```markdown
# SOUL.md - Who You Are

- **Name:** {agent.name}
- **Vibe:** 친근하고 도움이 되는 어시스턴트

Be genuinely helpful. Have opinions. Be resourceful.
```

**USER.md** (공통)
```markdown
# USER.md - About Your Human

- **Name:** (사용자가 입력)
- **Timezone:** Asia/Seoul
```

**IDENTITY.md** (각 에이전트별)
```markdown
# IDENTITY.md

- **Name:** {agent.name}
- **Creature:** AI Agent
- **Emoji:** (선택)
```

**TOOLS.md** (빈 파일)
```markdown
# TOOLS.md - Local Notes

Add your environment-specific notes here.
```

**MEMORY.md** (빈 파일)
```markdown
# MEMORY.md - Long-term Memory

(에이전트가 기억할 내용을 여기에 기록)
```

#### Step 3: Config 파일 생성/업데이트

`~/.openclaw/openclaw.json` 파일을 다음 형식으로 생성 또는 업데이트:

```json
{
  "agents": {
    "list": [
      {
        "id": "{agent1.id}",
        "workspace": "{agent1.workspace}"
      },
      {
        "id": "{agent2.id}",
        "workspace": "{agent2.workspace}"
      },
      {
        "id": "{agent3.id}",
        "workspace": "{agent3.workspace}"
      }
    ]
  },
  "models": {
    "profiles": {
      "{agent1.id}": {
        "default": "{agent1.model}"
      },
      "{agent2.id}": {
        "default": "{agent2.model}"
      },
      "{agent3.id}": {
        "default": "{agent3.model}"
      }
    }
  },
  "channels": {
    "telegram": {
      "accounts": {
        "{agent1.id}": {
          "botToken": "{agent1.telegram_token}"
        },
        "{agent2.id}": {
          "botToken": "{agent2.telegram_token}"
        },
        "{agent3.id}": {
          "botToken": "{agent3.telegram_token}"
        }
      }
    }
  },
  "bindings": [
    {
      "agentId": "{agent1.id}",
      "match": {
        "channel": "telegram",
        "accountId": "{agent1.id}"
      }
    },
    {
      "agentId": "{agent2.id}",
      "match": {
        "channel": "telegram",
        "accountId": "{agent2.id}"
      }
    },
    {
      "agentId": "{agent3.id}",
      "match": {
        "channel": "telegram",
        "accountId": "{agent3.id}"
      }
    }
  ]
}
```

#### Step 4: Gateway 재시작
```bash
openclaw gateway restart
```

#### Step 5: 검증
```bash
openclaw status
```

### 3. 완료 후 보고

설정 완료 후 다음 정보를 알려줘:
- [ ] 생성된 workspace 경로들
- [ ] 각 에이전트의 텔레그램 봇 username
- [ ] 테스트 방법

---

## [EXAMPLE] 실제 사용 예시

### 예시 1: 3명의 악마 에이전트

```yaml
agents:
  - id: "belphegor"
    name: "벨페고르"
    workspace: "/Users/username/clawd"
    telegram_token: "123456:ABC..."
    model: "anthropic/claude-sonnet-4"
    
  - id: "mammon"
    name: "마몬"
    workspace: "~/.openclaw/workspace-mammon"
    telegram_token: "234567:DEF..."
    model: "google/gemini-2.0-flash"
    
  - id: "asmodeus"
    name: "아스모데우스"
    workspace: "~/.openclaw/workspace-asmodeus"
    telegram_token: "345678:GHI..."
    model: "google/gemini-2.0-flash"
```

### 예시 2: 업무용/개인용 분리

```yaml
agents:
  - id: "work"
    name: "업무 어시스턴트"
    workspace: "~/.openclaw/workspace-work"
    telegram_token: "토큰1"
    model: "anthropic/claude-sonnet-4"
    
  - id: "personal"
    name: "개인 비서"
    workspace: "~/.openclaw/workspace-personal"
    telegram_token: "토큰2"
    model: "google/gemini-2.0-flash"
```

---

## [NOTES] 주의사항

1. **agents.list[] 형식 필수**: agents 밑에 직접 키로 넣으면 에러남
2. **토큰 보안**: 토큰은 절대 공개하지 말 것
3. **workspace 경로**: 틸드(~) 또는 절대경로 사용
4. **모델 설정**: Google OAuth 또는 API Key 필요
5. **bindings**: 에이전트-채널 연결 필수

---

## [QUICK] 빠른 설정 (복사해서 수정)

아래 YAML을 수정해서 에이전트에게 전달:

```yaml
# === 여기만 수정하세요 ===

user_name: "사용자이름"
user_timezone: "Asia/Seoul"

agents:
  - id: "agent1"
    name: "에이전트1"
    workspace: "~/.openclaw/workspace-agent1"
    telegram_token: "토큰입력"
    model: "anthropic/claude-sonnet-4"
    personality: "친절하고 전문적인"
    
  - id: "agent2"
    name: "에이전트2"
    workspace: "~/.openclaw/workspace-agent2"
    telegram_token: "토큰입력"
    model: "google/gemini-2.0-flash"
    personality: "유머러스하고 캐주얼한"

# === 수정 끝 ===

# 위 설정대로 멀티봇 환경을 구성해줘.
# 1. Workspace 폴더와 기본 파일 생성
# 2. ~/.openclaw/openclaw.json 업데이트
# 3. Gateway 재시작
# 4. 결과 보고
```

---

## [ADD] 기존 환경에 봇 추가하기

이미 멀티봇이 돌아가고 있는데 새 봇을 추가하고 싶을 때 사용하세요.

### 봇 추가 템플릿

```yaml
# === 추가할 에이전트 ===
new_agent:
  id: "newbot"
  name: "새 에이전트"
  workspace: "~/.openclaw/workspace-newbot"
  telegram_token: "새봇토큰"
  model: "google/gemini-2.0-flash"
  personality: "원하는 성격"

# === 작업 지시 ===
# 기존 멀티봇 환경에 위 에이전트를 추가해줘.
#
# 수행할 작업:
# 1. 새 workspace 폴더 생성
# 2. 기본 파일 생성 (AGENTS.md, SOUL.md, USER.md, IDENTITY.md, TOOLS.md, MEMORY.md)
# 3. 기존 ~/.openclaw/openclaw.json에 새 에이전트 추가 (기존 설정 유지!)
#    - agents.list[]에 추가
#    - channels.telegram.accounts에 추가
#    - bindings에 추가
# 4. Gateway 재시작
# 5. 결과 보고
```

### 추가 시 Config 변경 예시

기존 config:
```json
{
  "agents": {
    "list": [
      { "id": "agent1", "workspace": "..." },
      { "id": "agent2", "workspace": "..." }
    ]
  }
}
```

추가 후:
```json
{
  "agents": {
    "list": [
      { "id": "agent1", "workspace": "..." },
      { "id": "agent2", "workspace": "..." },
      { "id": "newbot", "workspace": "~/.openclaw/workspace-newbot" }  // 추가됨
    ]
  }
}
```

### 주의사항

1. **기존 설정 보존**: 새 에이전트만 추가하고 기존 설정은 건드리지 말 것
2. **ID 중복 확인**: 기존 에이전트와 같은 ID 사용 금지
3. **토큰 확인**: BotFather에서 새 봇 생성 후 토큰 받아오기

---

## [REMOVE] 봇 제거하기

에이전트를 제거하고 싶을 때:

```yaml
# === 제거할 에이전트 ===
remove_agent_id: "botid"

# === 작업 지시 ===
# 위 ID의 에이전트를 멀티봇 환경에서 제거해줘.
#
# 수행할 작업:
# 1. ~/.openclaw/openclaw.json에서 해당 에이전트 제거
#    - agents.list[]에서 제거
#    - channels.telegram.accounts에서 제거
#    - bindings에서 제거
# 2. Gateway 재시작
# 3. (선택) workspace 폴더는 보존 또는 삭제 확인
```

### 주의사항

- Workspace 폴더는 기본적으로 **삭제하지 않음** (대화 기록/메모리 보존)
- 완전 삭제 원하면 명시적으로 요청
