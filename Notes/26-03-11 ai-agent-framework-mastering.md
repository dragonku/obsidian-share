---
title: AI Agent Framework Mastering
created: 2026-03-11
updated: 2026-03-11
type: curriculum
status: in-progress
total_hours: 8
tags:
  - AI-Agent
  - LangChain
  - LangGraph
  - DeepAgents
  - MCP
  - A2A
  - 교육과정
aliases:
  - Agent Framework 교육
  - AI 에이전트 마스터링
---

# AI Agent Framework Mastering

> **목표**: LLM 기반 Agent 패러다임을 이해하고, LangChain → LangGraph → DeepAgents → Skills/MCP → A2A 순으로
> 실제 동작하는 "메일 모니터링 Agent"를 End-to-End로 구축한다.
>
> **총 소요**: 약 8시간 (6개 모듈)
> **사전 요구**: Python 3.12+, Anthropic API Key, 기본적인 CLI 경험
> **최종 산출물**: 메일 Inbox를 폴링하여 조건 필터 → 요약 → 알림하는 멀티에이전트 시스템

---

## 커리큘럼 전체 맵

| # | 모듈 | 유형 | 시간 | 핵심 키워드 |
|---|------|------|------|------------|
| 1 | Agent 패러다임 | 이론 | 1H | LLM 특성, ReAct, Tool Use, Framework→Runtime→Harness |
| 2 | LangChain/LangGraph Agent 구성 | 이론+실습 | 2H | Agent Loop, Tool 호출, 상태 관리, Long-Running |
| 3 | DeepAgents & Harness | 이론+실습 | 2H | create_deep_agent, Middleware, 폴링 구성 |
| 4 | Skills & MCP 연계 | 이론+실습 | 1H | 메일 Skill 설계, MCP 서버, Tool 정규화 |
| 5 | A2A로 역할 분리 | 이론+실습 | 1H | Agent Card, Task/Artifact, Check↔Notify |
| 6 | 통합 Demo & Wrap-up | 이론+실습 | 1H | E2E 데모, HITL 확장 포인트 |

---

## 환경 준비 (모든 모듈 공통)

### 프로젝트 구조

```
agent-mastering/
├── .env                    # API 키
├── requirements.txt
├── module1_paradigm/       # 이론 노트 + 간단 스크립트
├── module2_langchain/      # LangChain/LangGraph 실습
├── module3_deepagents/     # DeepAgents 실습
├── module4_skills_mcp/     # Skills + MCP 서버
├── module5_a2a/            # A2A 연동
└── module6_integration/    # 통합 데모
```

### 초기 설정

```bash
mkdir agent-mastering && cd agent-mastering
python -m venv .venv
source .venv/bin/activate

pip install langchain langchain-anthropic langgraph deepagents
pip install python-dotenv httpx uvicorn starlette
pip install a2a-python

mkdir module{1_paradigm,2_langchain,3_deepagents,4_skills_mcp,5_a2a,6_integration}

cat > .env << 'EOF'
ANTHROPIC_API_KEY=your-key-here
EOF
```

---

# Module 1: Agent 패러다임 (이론, 1H)

> **학습 목표**
> - LLM의 동작 방식과 프롬프트 특성을 이해한다
> - Agent의 정의와 ReAct 패턴을 설명할 수 있다
> - Framework / Runtime / Harness 계층 구분을 이해한다

---

## 1.1 LLM 동작 방식과 프롬프트 특성

LLM은 본질적으로 **다음 토큰 예측기**다. 이 단순한 메커니즘이 놀라운 능력을 보이지만, 근본적인 한계를 만든다.

**LLM이 잘하는 것:** 자연어 이해/생성, 패턴 인식, 코드 작성, 구조화된 출력

**LLM이 못하는 것:** 실시간 데이터 접근, 외부 시스템 조작, 정확한 수학 계산, 상태 유지

**프롬프트 특성:**
```
┌─────────────────────────────────────────────────┐
│  프롬프트 = System + User + Assistant 히스토리   │
│  ① 순서 민감: 앞의 지시가 뒤를 지배               │
│  ② 컨텍스트 윈도우: 유한한 작업 메모리 (200K 토큰) │
│  ③ 지시 추종: 구조화된 포맷 → 안정적 행동          │
│  ④ Few-shot 학습: 예시가 출력 품질을 결정           │
└─────────────────────────────────────────────────┘
```

---

## 1.2 Agent란? ReAct & Tool Use 개념

**Agent = LLM + 도구(Tools) + 루프(Loop)**

### ReAct 패턴 (Reason + Act)

```
사용자: "서울의 오늘 날씨를 알려줘"

┌─ Thought: 날씨를 알려면 현재 날씨 데이터가 필요하다.
├─ Action:  weather_api(city="Seoul")
├─ Observation: {"temp": 12, "condition": "맑음", "humidity": 45}
├─ Thought: 날씨 데이터를 받았다.
└─ Answer: "서울의 오늘 날씨는 12°C로 맑습니다."
```

**핵심**: LLM은 "어떤 도구를 언제 쓸지 판단"하고, 실제 실행은 Agent 런타임이 담당한다.

---

## 1.3 LLM → Agent → Agent Harness 진화

```
Stage 1: LLM 직접 호출
User → API Call → LLM → Response
문제: 도구 사용 불가, 상태 없음

Stage 2: Agent (Framework + Runtime)
User → Agent Loop → LLM (판단) + Tool 실행 + 상태 관리
문제: 루프/도구/프롬프트를 직접 구성해야 함

Stage 3: Agent Harness
User → Harness → Planning + Sub-agents + Filesystem + Tool 실행
장점: 배터리 포함, 즉시 동작
```

### Framework vs Runtime vs Harness

| 구분 | 대표 | 핵심 가치 | 비유 |
|------|------|----------|------|
| **Framework** | LangChain | 추상화, 컴포넌트 재사용 | 레고 블록 |
| **Runtime** | LangGraph | 실행 엔진, 상태 머신, 체크포인팅 | 블록을 조립하는 테이블 |
| **Harness** | DeepAgents | 즉시 동작, Planning/FS/SubAgent 내장 | 완성된 로봇 키트 |

```
┌─────────────────────────────┐
│     DeepAgents (Harness)     │
├─────────────────────────────┤
│     LangChain (Framework)    │
├─────────────────────────────┤
│     LangGraph (Runtime)      │
└─────────────────────────────┘
```

---

## 1.5 체크포인트

> - [ ] LLM의 한계 3가지를 설명할 수 있는가?
> - [ ] ReAct 패턴의 Thought → Action → Observation 흐름을 그릴 수 있는가?
> - [ ] Framework / Runtime / Harness의 차이를 설명할 수 있는가?

---

# Module 2: LangChain/LangGraph로 Agent 구성 (이론+실습, 2H)

> **학습 목표**
> - 단일 Agent Loop를 직접 구현한다
> - Tool 호출과 상태 관리를 이해한다
> - 상태 / 재시도 / 중단 제어를 직접 코딩한다
> - Long-Running Agent를 LangGraph로 구성한다

---

## 2.1 단일 Agent Loop 구현 (LangChain)

모든 Agent의 핵심 루프:
```
while True:
    1. LLM에게 현재 상태 + 도구 목록 전달
    2. LLM이 응답 생성 (텍스트 or tool_call)
    3. if tool_call → 도구 실행 → 결과를 메시지에 추가 → 1로
    4. if 텍스트 → 최종 응답 반환 → 루프 종료
```

### 실습 2-1: 순수 Agent Loop

> 개별 노트: [[26-03-11 실습2-1 순수 Agent Loop]]

`module2_langchain/01_agent_loop.py`:

```python
"""실습 2-1: LangChain으로 Agent Loop 직접 구현"""
from dotenv import load_dotenv
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage, ToolMessage
from langchain_core.tools import tool

load_dotenv()

@tool
def get_inbox_count() -> str:
    """메일 수신함의 읽지 않은 메일 수를 반환한다."""
    return "읽지 않은 메일: 5통"

@tool
def get_email_subject(index: int) -> str:
    """특정 인덱스의 메일 제목을 반환한다."""
    subjects = [
        "[긴급] 서버 장애 보고", "주간 회의 안건", "점심 메뉴 추천",
        "[PR] feature/auth 리뷰 요청", "구독 뉴스레터"
    ]
    if 0 <= index < len(subjects):
        return f"메일 #{index}: {subjects[index]}"
    return "해당 인덱스의 메일이 없습니다"

llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0)
tools = [get_inbox_count, get_email_subject]
llm_with_tools = llm.bind_tools(tools)

def run_agent(user_input: str, max_iterations: int = 10):
    messages = [HumanMessage(content=user_input)]
    tool_map = {t.name: t for t in tools}

    for i in range(max_iterations):
        print(f"\n--- Iteration {i+1} ---")
        response = llm_with_tools.invoke(messages)
        messages.append(response)

        if not response.tool_calls:
            print(f"[최종 응답] {response.content}")
            return response.content

        for tc in response.tool_calls:
            print(f"[Tool Call] {tc['name']}({tc['args']})")
            result = tool_map[tc["name"]].invoke(tc["args"])
            print(f"[Tool Result] {result}")
            messages.append(ToolMessage(content=str(result), tool_call_id=tc["id"]))

    return "최대 반복 횟수 초과"

if __name__ == "__main__":
    print("=" * 60)
    print("실습 2-1: 순수 Agent Loop")
    print("=" * 60)
    run_agent("수신함에 읽지 않은 메일이 몇 통 있는지 확인하고, 첫 번째 메일 제목을 알려줘")
```

---

## 2.2 Tool 호출과 상태 관리 (LangGraph)

LangGraph는 **StateGraph**로 흐름을 선언적으로 정의한다.

```
┌────────────┐  tool_call   ┌───────────┐
│   Agent    │ ───────────→ │   Tools   │
│  (reason)  │ ←─────────── │  (action) │
└─────┬──────┘  tool_result └───────────┘
      │ no tool_call
      ▼ [END]
```

### 실습 2-2: LangGraph Agent

> 개별 노트: [[26-03-11 실습2-2 LangGraph 상태 머신]]

`module2_langchain/02_langgraph_agent.py`:

```python
"""실습 2-2: LangGraph로 동일한 Agent를 상태 머신으로 구현"""
from dotenv import load_dotenv
from typing import Annotated, TypedDict
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import AnyMessage
from langchain_core.tools import tool
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode

load_dotenv()

class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    iteration_count: int

@tool
def get_inbox_count() -> str:
    """메일 수신함의 읽지 않은 메일 수를 반환한다."""
    return "읽지 않은 메일: 5통"

@tool
def get_email_subject(index: int) -> str:
    """특정 인덱스의 메일 제목을 반환한다."""
    subjects = [
        "[긴급] 서버 장애 보고", "주간 회의 안건", "점심 메뉴 추천",
        "[PR] feature/auth 리뷰 요청", "구독 뉴스레터"
    ]
    if 0 <= index < len(subjects):
        return f"메일 #{index}: {subjects[index]}"
    return "해당 인덱스의 메일이 없습니다"

tools = [get_inbox_count, get_email_subject]
llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0)
llm_with_tools = llm.bind_tools(tools)

def agent_node(state: AgentState) -> dict:
    response = llm_with_tools.invoke(state["messages"])
    count = state.get("iteration_count", 0) + 1
    print(f"[Agent Node] Iteration {count}, tool_calls: {len(response.tool_calls)}")
    return {"messages": [response], "iteration_count": count}

tool_node = ToolNode(tools)

def should_continue(state: AgentState) -> str:
    if state["messages"][-1].tool_calls:
        return "tools"
    return END

workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue)
workflow.add_edge("tools", "agent")
app = workflow.compile()

if __name__ == "__main__":
    print("=" * 60)
    print("실습 2-2: LangGraph Agent (상태 머신)")
    print("=" * 60)
    result = app.invoke({
        "messages": [("user", "수신함 확인하고 첫 번째와 네 번째 메일 제목 알려줘")],
        "iteration_count": 0
    })
    print(f"\n[최종 응답] {result['messages'][-1].content}")
    print(f"[총 반복] {result['iteration_count']}회")
```

---

## 2.3 상태, 재시도, 중단 제어

### 실습 2-3: 에러 핸들링 & 재시도

> 개별 노트: [[26-03-11 실습2-3 에러 핸들링과 재시도]]

`module2_langchain/03_retry_control.py`:

```python
"""실습 2-3: 실패하는 도구를 포함한 Agent — 재시도/중단 패턴"""
import random
from dotenv import load_dotenv
from typing import Annotated, TypedDict
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import AnyMessage, ToolMessage
from langchain_core.tools import tool
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages

load_dotenv()

class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    retry_count: int
    max_retries: int

@tool
def fetch_email_content(email_id: int) -> str:
    """메일 본문을 가져온다. 네트워크 오류가 발생할 수 있다."""
    if random.random() < 0.5:
        raise Exception(f"IMAP 연결 타임아웃 (email_id={email_id})")
    return f"[메일 #{email_id}] 내일 오전 10시 배포 미팅이 있습니다."

def safe_tool_node(state: AgentState) -> dict:
    last_msg = state["messages"][-1]
    results = []
    for tc in last_msg.tool_calls:
        try:
            result = fetch_email_content.invoke(tc["args"])
            results.append(ToolMessage(content=str(result), tool_call_id=tc["id"]))
        except Exception as e:
            retry = state.get("retry_count", 0)
            if retry < state.get("max_retries", 3):
                results.append(ToolMessage(
                    content=f"[ERROR] {str(e)} — 재시도 {retry+1}/{state['max_retries']}",
                    tool_call_id=tc["id"]
                ))
                return {"messages": results, "retry_count": retry + 1}
            else:
                results.append(ToolMessage(
                    content=f"[FAILED] 최대 재시도 초과. 마지막 오류: {str(e)}",
                    tool_call_id=tc["id"]
                ))
    return {"messages": results}

llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0)
llm_with_tools = llm.bind_tools([fetch_email_content])

def agent_node(state: AgentState):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

def route(state: AgentState) -> str:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return END

workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", safe_tool_node)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", route)
workflow.add_edge("tools", "agent")
app = workflow.compile()

if __name__ == "__main__":
    print("=" * 60)
    print("실습 2-3: 재시도/중단 제어")
    print("=" * 60)
    result = app.invoke({
        "messages": [("user", "메일 #2의 본문을 가져와줘")],
        "retry_count": 0,
        "max_retries": 3
    })
    print(f"\n[최종] {result['messages'][-1].content}")
```

---

## 2.4 Long-Running Agent 구성

### 실습 2-4: 체크포인팅과 Long-Running 패턴

> 개별 노트: [[26-03-11 실습2-4 체크포인팅 Long-Running Agent]]

`module2_langchain/04_long_running.py`:

```python
"""실습 2-4: LangGraph 체크포인팅을 활용한 Long-Running Agent"""
import time
from dotenv import load_dotenv
from typing import Annotated, TypedDict
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import AnyMessage
from langchain_core.tools import tool
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import ToolNode

load_dotenv()

class MonitorState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    check_count: int
    last_check_time: str
    found_urgent: bool

@tool
def check_new_emails() -> str:
    """새 메일을 확인한다. (시뮬레이션)"""
    import random
    now = time.strftime("%H:%M:%S")
    if random.random() < 0.3:
        return f"[{now}] 🚨 긴급 메일 발견: [장애] 프로덕션 API 응답 지연"
    return f"[{now}] 새 메일 없음"

llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0)
llm_with_tools = llm.bind_tools([check_new_emails])

def agent_node(state: MonitorState):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

def tool_node_fn(state: MonitorState):
    node = ToolNode([check_new_emails])
    result = node.invoke(state)
    count = state.get("check_count", 0) + 1
    last_msg = result["messages"][-1].content if result["messages"] else ""
    return {
        **result,
        "check_count": count,
        "last_check_time": time.strftime("%H:%M:%S"),
        "found_urgent": "긴급" in last_msg or state.get("found_urgent", False)
    }

def route(state: MonitorState) -> str:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return END

checkpointer = MemorySaver()
workflow = StateGraph(MonitorState)
workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node_fn)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", route)
workflow.add_edge("tools", "agent")
app = workflow.compile(checkpointer=checkpointer)

if __name__ == "__main__":
    print("=" * 60)
    print("실습 2-4: Long-Running Agent (체크포인팅)")
    print("=" * 60)
    thread_config = {"configurable": {"thread_id": "mail-monitor-001"}}

    print("\n[Run 1] 첫 번째 체크")
    result = app.invoke(
        {"messages": [("user", "새 메일 확인해줘")],
         "check_count": 0, "last_check_time": "", "found_urgent": False},
        config=thread_config
    )
    print(f"  응답: {result['messages'][-1].content[:100]}...")
    print(f"  체크 횟수: {result['check_count']}")

    time.sleep(2)
    print("\n[Run 2] 이어서 체크 (상태 유지)")
    result = app.invoke({"messages": [("user", "다시 확인해줘")]}, config=thread_config)
    print(f"  누적 체크 횟수: {result['check_count']}")
    print(f"  긴급 메일 발견: {result['found_urgent']}")
```

---

## 2.5 체크포인트

> - [ ] 실습 2-1: 수동 Agent Loop가 정상 동작하는가?
> - [ ] 실습 2-2: LangGraph의 상태가 iteration_count를 정확히 추적하는가?
> - [ ] 실습 2-3: 도구 실패 시 재시도가 동작하고, max_retries 초과 시 중단하는가?
> - [ ] 실습 2-4: 두 번째 invoke에서 check_count가 누적되는가?

---

# Module 3: DeepAgents & Harness (이론+실습, 2H)

> **학습 목표**
> - Agent / Runtime / Harness 관계를 코드로 체감한다
> - DeepAgents의 Middleware 아키텍처를 이해한다
> - Module 2와 동일한 기능을 DeepAgents로 구현하여 비교한다
> - 스케줄/폴링 패턴을 구성한다

---

## 3.1 DeepAgents 핵심 개념

DeepAgents가 기본 제공하는 4가지:
```
┌─────────────────────────────────────────┐
│  1. Planning Tool (TodoListMiddleware)   │
│  2. Filesystem (FilesystemMiddleware)    │
│  3. Sub-agents (SubAgentMiddleware)      │
│  4. Detailed Prompt                      │
└─────────────────────────────────────────┘
```

### Middleware 스택
```
┌──────────────────────────────────┐
│  SummarizationMiddleware         │
├──────────────────────────────────┤
│  TodoListMiddleware              │
├──────────────────────────────────┤
│  FilesystemMiddleware            │
├──────────────────────────────────┤
│  SubAgentMiddleware              │
├──────────────────────────────────┤
│  [사용자 커스텀 Middleware]       │
├──────────────────────────────────┤
│  LangGraph Runtime               │
└──────────────────────────────────┘
```

---

## 3.2 DeepAgents vs LangGraph 비교

| 관점 | LangGraph | DeepAgents |
|------|-----------|------------|
| 도구 등록 | `llm.bind_tools([...])` | `create_deep_agent(tools=[...])` |
| Agent Loop | 직접 StateGraph 구성 | 자동 내장 |
| Planning | 직접 구현 | TodoListMiddleware 내장 |
| 코드량 | ~80줄 | ~15줄 |

---

## 3.3 실습: DeepAgents로 메일 Agent 구현

### 실습 3-1: 기본 DeepAgent 생성

> 개별 노트: [[26-03-11 실습3-1 기본 DeepAgent]]

`module3_deepagents/01_basic_deep_agent.py`:

```python
"""실습 3-1: DeepAgents로 메일 확인 Agent — Module 2의 80줄을 ~15줄로"""
from dotenv import load_dotenv
from langchain_core.tools import tool
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

load_dotenv()

@tool
def get_inbox_count() -> str:
    """메일 수신함의 읽지 않은 메일 수를 반환한다."""
    return "읽지 않은 메일: 5통"

@tool
def get_email_subject(index: int) -> str:
    """특정 인덱스의 메일 제목을 반환한다."""
    subjects = [
        "[긴급] 서버 장애 보고", "주간 회의 안건", "점심 메뉴 추천",
        "[PR] feature/auth 리뷰 요청", "구독 뉴스레터"
    ]
    if 0 <= index < len(subjects):
        return f"메일 #{index}: {subjects[index]}"
    return "해당 인덱스의 메일이 없습니다"

agent = create_deep_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-20250514"),
    tools=[get_inbox_count, get_email_subject],
    system_prompt="당신은 메일 수신함을 모니터링하는 에이전트입니다."
)

if __name__ == "__main__":
    print("=" * 60)
    print("실습 3-1: DeepAgents 기본 Agent")
    print("=" * 60)
    result = agent.invoke({
        "messages": [{"role": "user", "content": "수신함 확인하고 긴급 메일이 있는지 제목들을 확인해줘"}]
    })
    for msg in reversed(result["messages"]):
        if hasattr(msg, "content") and isinstance(msg.content, str) and msg.content:
            print(f"\n[응답]\n{msg.content}")
            break
```

---

## 3.4 실습: 스케줄/폴링 구성

### 실습 3-2: 반복 폴링 Agent

> 개별 노트: [[26-03-11 실습3-2 반복 폴링 Agent]]

`module3_deepagents/02_polling_agent.py`:

```python
"""실습 3-2: 주기적으로 메일을 체크하는 폴링 Agent"""
import asyncio
import time
from dotenv import load_dotenv
from langchain_core.tools import tool
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

load_dotenv()

class MailSimulator:
    def __init__(self):
        self.check_count = 0
        self.emails = [
            {"id": 1, "subject": "점심 뭐 먹을까", "urgent": False, "read": False},
            {"id": 2, "subject": "[장애] DB 커넥션 풀 고갈", "urgent": True, "read": False},
        ]

    def check(self) -> str:
        self.check_count += 1
        unread = [e for e in self.emails if not e["read"]]
        if not unread:
            return f"[Check #{self.check_count}] 새 메일 없음"
        result = f"[Check #{self.check_count}] 읽지 않은 메일 {len(unread)}통:\n"
        for e in unread:
            flag = "🚨" if e["urgent"] else "📧"
            result += f"  {flag} #{e['id']}: {e['subject']}\n"
        return result.strip()

    def mark_read(self, email_id: int) -> str:
        for e in self.emails:
            if e["id"] == email_id:
                e["read"] = True
                return f"메일 #{email_id} 읽음 처리 완료"
        return f"메일 #{email_id}를 찾을 수 없습니다"

mail_sim = MailSimulator()

@tool
def check_inbox() -> str:
    """수신함의 읽지 않은 메일을 확인한다."""
    return mail_sim.check()

@tool
def mark_as_read(email_id: int) -> str:
    """특정 메일을 읽음 처리한다."""
    return mail_sim.mark_read(email_id)

agent = create_deep_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-20250514"),
    tools=[check_inbox, mark_as_read],
    system_prompt="""당신은 메일 모니터링 에이전트입니다.
규칙:
- 새 메일을 확인하고 결과를 보고한다
- 긴급(🚨) 메일이 있으면 반드시 강조한다
- 확인한 메일은 읽음 처리한다"""
)

async def polling_loop(interval_sec: int = 5, max_checks: int = 3):
    for i in range(max_checks):
        print(f"\n{'='*50}")
        print(f"[Polling {i+1}/{max_checks}] {time.strftime('%H:%M:%S')}")
        print(f"{'='*50}")
        result = agent.invoke({
            "messages": [{"role": "user", "content": "새 메일 확인하고 긴급 메일이 있으면 알려줘. 확인한 메일은 읽음 처리해줘."}]
        })
        for msg in reversed(result["messages"]):
            if hasattr(msg, "content") and isinstance(msg.content, str) and msg.content:
                print(f"\n{msg.content}")
                break
        if i < max_checks - 1:
            print(f"\n⏰ {interval_sec}초 후 다음 체크...")
            await asyncio.sleep(interval_sec)
    print(f"\n✅ 폴링 완료. 총 {mail_sim.check_count}회 체크")

if __name__ == "__main__":
    asyncio.run(polling_loop(interval_sec=3, max_checks=3))
```

---

## 3.5 체크포인트

> - [ ] 실습 3-1: DeepAgent가 자동으로 Planning → Tool 호출 → 보고를 수행하는가?
> - [ ] 실습 3-2: 폴링 루프가 주기적으로 동작하고, 읽음 처리로 상태가 변하는가?
> - [ ] Module 2와 코드량/복잡도 차이를 체감했는가?

---

# Module 4: Skills & MCP 연계 (이론+실습, 1H)

> **학습 목표**
> - Skill 개념과 MCP(Model Context Protocol)를 이해한다
> - 메일 Skills을 설계한다: Inbox 확인 / 조건 필터 / 요약
> - MCP 서버로 메일 API를 연결한다
> - Tool(MCP) 호출 결과를 Skill이 정규화/해석하는 패턴을 익힌다

---

## 4.1 Skills과 MCP 개념

```
Tool (단일 함수)     vs    Skill (역량 패키지)
check_inbox()              EmailMonitorSkill:
                             ├── check_inbox()
                             ├── filter_by_condition()
                             ├── summarize_email()
                             └── 후처리: 결과 정규화

MCP: Agent ←→ MCP Client ←→ MCP Server ←→ External Service
```

---

## 4.2 메일 Skills 설계

### 실습 4-1: Email Skill 모듈

> 개별 노트: [[26-03-11 실습4-1 Email Skill 모듈]]

`module4_skills_mcp/email_skill.py`:

```python
"""실습 4-1: 메일 모니터링 Skill 설계"""
from dataclasses import dataclass, field
from typing import Optional
from langchain_core.tools import tool

@dataclass
class EmailRecord:
    id: int
    subject: str
    sender: str
    received_at: str
    is_urgent: bool = False
    summary: Optional[str] = None
    tags: list[str] = field(default_factory=list)

MOCK_EMAILS = [
    EmailRecord(1, "[긴급] 프로덕션 API 500 에러 급증", "ops-alert@company.com",
                "2026-03-11 09:00", True, tags=["장애", "프로덕션"]),
    EmailRecord(2, "Weekly Sprint Review 안건", "pm@company.com",
                "2026-03-11 09:30", tags=["회의"]),
    EmailRecord(3, "PR #1234: auth 모듈 리팩토링", "dev@company.com",
                "2026-03-11 10:00", tags=["PR", "코드리뷰"]),
    EmailRecord(4, "오늘 점심 치킨 어때요?", "colleague@company.com",
                "2026-03-11 10:15", tags=["잡담"]),
    EmailRecord(5, "[긴급] 보안 패치 필요 - CVE-2026-XXXX", "security@company.com",
                "2026-03-11 10:30", True, tags=["보안", "긴급"]),
]

@tool
def check_inbox(limit: int = 10) -> str:
    """수신함의 메일 목록을 반환한다."""
    emails = MOCK_EMAILS[:limit]
    lines = [f"📬 수신함: {len(emails)}통"]
    for e in emails:
        flag = "🚨" if e.is_urgent else "📧"
        lines.append(f"  {flag} #{e.id} | {e.sender} | {e.subject}")
    return "\n".join(lines)

@tool
def filter_emails(condition: str) -> str:
    """조건에 따라 메일을 필터링한다. condition: 'urgent' | 'tag:XX' | 'from:XX'"""
    if condition == "urgent":
        results = [e for e in MOCK_EMAILS if e.is_urgent]
    elif condition.startswith("tag:"):
        tag = condition.split(":")[1]
        results = [e for e in MOCK_EMAILS if tag in e.tags]
    elif condition.startswith("from:"):
        kw = condition.split(":")[1]
        results = [e for e in MOCK_EMAILS if kw in e.sender]
    else:
        return f"알 수 없는 조건: {condition}"
    if not results:
        return f"조건 '{condition}'에 해당하는 메일 없음"
    lines = [f"🔍 필터 결과 ({condition}): {len(results)}통"]
    for e in results:
        lines.append(f"  #{e.id} | {e.subject} | {e.received_at}")
    return "\n".join(lines)

@tool
def summarize_email(email_id: int) -> str:
    """특정 메일의 요약을 반환한다."""
    summaries = {
        1: "프로덕션 API에서 500 에러가 급증 중. 즉시 확인 필요. 영향 범위: 결제 서비스",
        2: "이번 주 스프린트 리뷰 안건: 인증 모듈 완료, 대시보드 80% 진행",
        3: "auth 모듈을 JWT에서 OAuth2로 전환하는 PR. 리뷰어 2명 필요",
        4: "점심 메뉴 제안. 업무 관련 아님",
        5: "외부 라이브러리 CVE 발견. 패치 버전 3.2.1로 업그레이드 필요. 긴급도: Critical",
    }
    for e in MOCK_EMAILS:
        if e.id == email_id:
            return f"📋 메일 #{e.id} 요약:\n제목: {e.subject}\n발신: {e.sender}\n내용: {summaries.get(e.id, '요약 없음')}"
    return f"메일 #{email_id}를 찾을 수 없습니다"

EMAIL_SKILL = {
    "name": "EmailMonitorSkill",
    "description": "메일 수신함 모니터링: 확인, 필터, 요약",
    "tools": [check_inbox, filter_emails, summarize_email],
    "trigger_keywords": ["메일", "이메일", "수신함", "inbox", "긴급"],
}
```

---

## 4.3 MCP 서버로 메일 API 연결

### 실습 4-2: 간이 MCP 서버

> 개별 노트: [[26-03-11 실습4-2 간이 MCP 서버]]

`module4_skills_mcp/mcp_mail_server.py`:

```python
"""실습 4-2: MCP 서버 패턴으로 메일 API 노출 (port 8100)"""
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.routing import Route
import json, uvicorn
from email_skill import MOCK_EMAILS

TOOL_DEFINITIONS = [
    {"name": "check_inbox", "description": "수신함의 메일 목록을 반환한다",
     "inputSchema": {"type": "object", "properties": {"limit": {"type": "integer", "default": 10}}}},
    {"name": "filter_emails", "description": "조건에 따라 메일을 필터링한다",
     "inputSchema": {"type": "object", "properties": {"condition": {"type": "string"}}, "required": ["condition"]}},
    {"name": "summarize_email", "description": "특정 메일의 요약을 반환한다",
     "inputSchema": {"type": "object", "properties": {"email_id": {"type": "integer"}}, "required": ["email_id"]}},
]

def execute_tool(name: str, args: dict) -> str:
    if name == "check_inbox":
        emails = MOCK_EMAILS[:args.get("limit", 10)]
        return json.dumps([{"id": e.id, "subject": e.subject, "sender": e.sender,
                            "is_urgent": e.is_urgent, "tags": e.tags} for e in emails], ensure_ascii=False)
    elif name == "filter_emails":
        condition = args["condition"]
        results = [e for e in MOCK_EMAILS if e.is_urgent] if condition == "urgent" else MOCK_EMAILS
        return json.dumps([{"id": e.id, "subject": e.subject, "is_urgent": e.is_urgent}
                           for e in results], ensure_ascii=False)
    elif name == "summarize_email":
        eid = args["email_id"]
        for e in MOCK_EMAILS:
            if e.id == eid:
                return json.dumps({"id": e.id, "subject": e.subject,
                                   "summary": f"{e.subject}에 대한 요약 내용"}, ensure_ascii=False)
        return json.dumps({"error": f"메일 #{eid} 없음"})
    return json.dumps({"error": f"알 수 없는 도구: {name}"})

async def tools_list(request):
    return JSONResponse({"tools": TOOL_DEFINITIONS})

async def tools_call(request):
    body = await request.json()
    result = execute_tool(body.get("name"), body.get("arguments", {}))
    return JSONResponse({"content": [{"type": "text", "text": result}]})

app = Starlette(routes=[
    Route("/mcp/tools/list", tools_list, methods=["GET"]),
    Route("/mcp/tools/call", tools_call, methods=["POST"]),
])

if __name__ == "__main__":
    print("🔌 MCP 메일 서버 시작: http://localhost:8100")
    uvicorn.run(app, host="0.0.0.0", port=8100)
```

---

## 4.4 Skill이 MCP 결과를 정규화/해석

### 실습 4-3: Skill + MCP 클라이언트 통합

> 개별 노트: [[26-03-11 실습4-3 Skill MCP 통합]]

`module4_skills_mcp/skill_with_mcp.py`:

```python
"""
실습 4-3: Skill이 MCP 서버를 호출하고 결과를 정규화
실행 순서:
  터미널 1: python mcp_mail_server.py
  터미널 2: python skill_with_mcp.py
"""
import httpx, json
from langchain_core.tools import tool
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

MCP_BASE = "http://localhost:8100"

def call_mcp_tool(name: str, arguments: dict = {}) -> dict:
    response = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": name, "arguments": arguments})
    return json.loads(response.json()["content"][0]["text"])

@tool
def mcp_check_inbox(limit: int = 10) -> str:
    """MCP 서버를 통해 메일을 확인하고 정규화된 보고서를 생성한다."""
    raw = call_mcp_tool("check_inbox", {"limit": limit})
    urgent = [e for e in raw if e.get("is_urgent")]
    normal = [e for e in raw if not e.get("is_urgent")]
    report = f"📬 메일 보고서 (총 {len(raw)}통)\n"
    if urgent:
        report += f"\n🚨 긴급 ({len(urgent)}통):\n"
        for e in urgent:
            report += f"  - [{e['id']}] {e['subject']} (from: {e['sender']})\n"
    report += f"\n📧 일반 ({len(normal)}통):\n"
    for e in normal:
        report += f"  - [{e['id']}] {e['subject']}\n"
    return report

@tool
def mcp_get_urgent_summary() -> str:
    """긴급 메일만 필터링하고 각각의 요약을 생성한다."""
    urgent_emails = call_mcp_tool("filter_emails", {"condition": "urgent"})
    if not urgent_emails:
        return "✅ 긴급 메일 없음"
    summaries = []
    for e in urgent_emails:
        detail = call_mcp_tool("summarize_email", {"email_id": e["id"]})
        summaries.append(f"🚨 [{e['id']}] {e['subject']}\n   → {detail.get('summary', 'N/A')}")
    return "긴급 메일 요약:\n" + "\n".join(summaries)

agent = create_deep_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-20250514"),
    tools=[mcp_check_inbox, mcp_get_urgent_summary],
    system_prompt="MCP 서버를 통해 메일 데이터를 가져옵니다. 항상 긴급 메일을 우선 확인합니다."
)

if __name__ == "__main__":
    print("=" * 60)
    print("실습 4-3: Skill + MCP 통합 Agent")
    print("=" * 60)
    result = agent.invoke({"messages": [{"role": "user", "content": "수신함을 확인하고 긴급 메일이 있으면 요약해줘"}]})
    for msg in reversed(result["messages"]):
        if hasattr(msg, "content") and isinstance(msg.content, str) and msg.content:
            print(f"\n{msg.content}")
            break
```

---

## 4.5 체크포인트

> - [ ] Tool과 Skill의 차이를 설명할 수 있는가?
> - [ ] MCP 서버가 정상 기동되고, `/mcp/tools/list`에서 도구 목록이 반환되는가?
> - [ ] Skill이 MCP raw 데이터를 정규화하여 Agent에게 전달하는 흐름을 이해했는가?

---

# Module 5: A2A로 역할 분리 (이론+실습, 1H)

> **학습 목표**
> - A2A(Agent-to-Agent) 프로토콜의 핵심 개념을 이해한다
> - Agent Card / Task / Artifact 구조를 파악한다
> - Check Agent ↔ Notify Agent 2개를 A2A로 연동한다

---

## 5.1 A2A 개념

```
MCP: Agent ↔ Tool (도구 호출)
A2A: Agent ↔ Agent (에이전트 간 협업)

[Check Agent]                    [Notify Agent]
     │  1. Agent Card 조회           │
     │  ─────────────────────────→   │
     │  ←─────────────────────────   │
     │  2. Task 전송 (메일 결과)      │
     │  ─────────────────────────→   │  3. 처리 (알림 생성)
     │  ←─────────────────────────   │
     │     (Task 완료 + Artifact)    │
```

**A2A 핵심 3요소**: Agent Card (명함), Task (작업 단위), Artifact (결과물)

---

## 5.2 실습: Check Agent ↔ Notify Agent 연동

### 실습 5-1: Notify Agent (A2A Server)

> 개별 노트: [[26-03-11 실습5-1 Notify Agent A2A Server]]

`module5_a2a/notify_agent_server.py`:

```python
"""실습 5-1: Notify Agent — A2A Server (port 8201)"""
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.routing import Route
import uuid
from datetime import datetime
import uvicorn

AGENT_CARD = {
    "name": "NotifyAgent",
    "description": "메일 체크 결과를 받아 알림 메시지를 생성하는 에이전트",
    "url": "http://localhost:8201",
    "version": "1.0.0",
    "capabilities": {"streaming": False, "pushNotifications": False},
    "authentication": {"schemes": []},
    "defaultInputModes": ["text/plain"],
    "defaultOutputModes": ["text/plain"],
    "skills": [{
        "id": "generate_notification",
        "name": "알림 생성",
        "description": "메일 체크 결과를 받아 팀 알림 메시지를 생성한다",
        "tags": ["notification", "alert"],
        "examples": ["긴급 메일 2통에 대한 알림을 생성해줘"]
    }]
}

tasks = {}

def process_notification(message_text: str) -> str:
    now = datetime.now().strftime("%H:%M")
    if "긴급" in message_text or "🚨" in message_text:
        return f"""🔔 [긴급 알림] {now}
━━━━━━━━━━━━━━━━━━━
{message_text}
━━━━━━━━━━━━━━━━━━━
⚡ 즉시 확인이 필요합니다!
📌 담당자를 멘션하여 전달하세요."""
    else:
        return f"""📋 [메일 리포트] {now}
{message_text}
─ 정기 체크 완료"""

async def agent_card(request):
    return JSONResponse(AGENT_CARD)

async def message_send(request):
    body = await request.json()
    params = body.get("params", {})
    message = params.get("message", {})
    parts = message.get("parts", [])
    user_text = next((p.get("text", "") for p in parts if p.get("kind") == "text"), "")

    task_id = str(uuid.uuid4())[:8]
    notification = process_notification(user_text)
    task = {
        "id": task_id,
        "status": {"state": "completed"},
        "artifacts": [{"parts": [{"kind": "text", "text": notification}]}]
    }
    tasks[task_id] = task
    print(f"[NotifyAgent] Task {task_id} 처리 완료")
    return JSONResponse({"jsonrpc": "2.0", "id": body.get("id"), "result": task})

app = Starlette(routes=[
    Route("/.well-known/agent.json", agent_card, methods=["GET"]),
    Route("/a2a", message_send, methods=["POST"]),
])

if __name__ == "__main__":
    print("🔔 Notify Agent (A2A Server) 시작: http://localhost:8201")
    uvicorn.run(app, host="0.0.0.0", port=8201)
```

### 실습 5-2: Check Agent (A2A Client)

> 개별 노트: [[26-03-11 실습5-2 Check Agent A2A Client]]

`module5_a2a/check_agent_client.py`:

```python
"""
실습 5-2: Check Agent — A2A Client
실행 순서:
  터미널 1: python notify_agent_server.py
  터미널 2: python mcp_mail_server.py
  터미널 3: python check_agent_client.py
"""
import httpx, json, uuid

NOTIFY_AGENT_URL = "http://localhost:8201"
MCP_BASE = "http://localhost:8100"

def discover_agent(base_url: str) -> dict:
    card = httpx.get(f"{base_url}/.well-known/agent.json").json()
    print(f"✅ Agent 발견: {card['name']}")
    print(f"   Skills: {[s['name'] for s in card['skills']]}")
    return card

def check_mail_via_mcp() -> str:
    resp = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": "check_inbox", "arguments": {"limit": 10}})
    emails = json.loads(resp.json()["content"][0]["text"])
    resp2 = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": "filter_emails", "arguments": {"condition": "urgent"}})
    urgent = json.loads(resp2.json()["content"][0]["text"])
    report = f"📬 수신함: 총 {len(emails)}통, 긴급 {len(urgent)}통\n"
    if urgent:
        report += "\n🚨 긴급 메일:\n"
        for e in urgent:
            report += f"  - [{e['id']}] {e['subject']}\n"
    return report

def send_to_notify_agent(agent_url: str, mail_report: str) -> dict:
    payload = {
        "jsonrpc": "2.0",
        "id": str(uuid.uuid4())[:8],
        "method": "message/send",
        "params": {"message": {"role": "user", "parts": [{"kind": "text", "text": mail_report}]}}
    }
    return httpx.post(f"{agent_url}/a2a", json=payload).json().get("result", {})

if __name__ == "__main__":
    print("=" * 60)
    print("실습 5-2: Check Agent → Notify Agent (A2A)")
    print("=" * 60)

    print("\n[Step 1] Agent Card 조회")
    card = discover_agent(NOTIFY_AGENT_URL)

    print("\n[Step 2] MCP로 메일 체크")
    report = check_mail_via_mcp()
    print(report)

    print("[Step 3] A2A로 Notify Agent에 전달")
    result = send_to_notify_agent(NOTIFY_AGENT_URL, report)
    print(f"\n[결과] Task 상태: {result.get('status', {}).get('state')}")
    artifacts = result.get("artifacts", [])
    if artifacts:
        for part in artifacts[0].get("parts", []):
            if part.get("kind") == "text":
                print(f"\n{part['text']}")
```

---

## 5.3 체크포인트

> - [ ] MCP와 A2A의 차이를 한 문장으로 설명할 수 있는가?
> - [ ] Agent Card에 필수적으로 들어가야 하는 정보를 나열할 수 있는가?
> - [ ] Check Agent → Notify Agent 연동이 정상 동작하는가?

---

# Module 6: 통합 Demo & Wrap-up (이론+실습, 1H)

> **학습 목표**
> - Module 1~5의 모든 요소를 하나의 End-to-End 파이프라인으로 통합한다
> - HITL(Human-in-the-Loop) 확장 포인트를 식별한다

---

## 6.1 통합 아키텍처

```
┌──────────────────────────────────────────────────────────┐
│  [Scheduler]                                              │
│      ↓                                                    │
│  [DeepAgent + EmailSkill]                                 │
│      ├─ MCP ──→ [MCP Mail Server :8100] → IMAP/Gmail     │
│      ├─ 분석 → 긴급 메일 필터 + 요약                      │
│      ├─ A2A ──→ [Notify Agent :8201]                      │
│      └─ (확장) → Slack/Teams 전송                         │
│                                                           │
│  HITL 포인트:                                             │
│  ① 메일 필터 조건을 사용자가 동적으로 변경                  │
│  ② 긴급 판별 결과를 사용자가 승인/거부                     │
│  ③ 알림 전송 전 사용자 확인 (interrupt)                    │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 통합 데모 스크립트

### 실습 6-1: End-to-End 데모

> 개별 노트: [[26-03-11 실습6-1 End-to-End 데모]]

`module6_integration/e2e_demo.py`:

```python
"""
실습 6-1: 메일 Agent End-to-End 데모

전제 조건:
  터미널 1: cd module4_skills_mcp && python mcp_mail_server.py
  터미널 2: cd module5_a2a && python notify_agent_server.py
  터미널 3: cd module6_integration && python e2e_demo.py
"""
import httpx, json, time, uuid

MCP_BASE = "http://localhost:8100"
A2A_NOTIFY_URL = "http://localhost:8201"

def banner(text):
    print(f"\n{'='*60}\n  {text}\n{'='*60}")

def phase1_check_mail():
    banner("Phase 1: MCP로 메일 수집")
    resp = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": "check_inbox", "arguments": {"limit": 10}})
    emails = json.loads(resp.json()["content"][0]["text"])
    print(f"📬 총 {len(emails)}통 수신")

    resp2 = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": "filter_emails", "arguments": {"condition": "urgent"}})
    urgent = json.loads(resp2.json()["content"][0]["text"])
    print(f"🚨 긴급 {len(urgent)}통 감지")

    summaries = []
    for e in urgent:
        resp3 = httpx.post(f"{MCP_BASE}/mcp/tools/call", json={"name": "summarize_email", "arguments": {"email_id": e["id"]}})
        detail = json.loads(resp3.json()["content"][0]["text"])
        summaries.append(detail)
        print(f"  📋 #{e['id']}: {detail.get('summary', 'N/A')}")

    return {"all_count": len(emails), "urgent": urgent, "summaries": summaries}

def phase2_generate_report(data):
    banner("Phase 2: 보고서 생성")
    report = f"""📬 메일 모니터링 보고서
━━━━━━━━━━━━━━━━━━━━━━━
시각: {time.strftime('%Y-%m-%d %H:%M:%S')}
총 메일: {data['all_count']}통
긴급 메일: {len(data['urgent'])}통
"""
    if data["urgent"]:
        report += "\n🚨 긴급 메일 상세:\n"
        for s in data["summaries"]:
            report += f"  [{s['id']}] {s['subject']}\n"
            report += f"      → {s.get('summary', 'N/A')}\n"
    print(report)
    return report

def phase3_notify_via_a2a(report):
    banner("Phase 3: A2A로 Notify Agent 호출")
    card = httpx.get(f"{A2A_NOTIFY_URL}/.well-known/agent.json").json()
    print(f"🤝 연결: {card['name']} v{card['version']}")

    payload = {
        "jsonrpc": "2.0",
        "id": str(uuid.uuid4())[:8],
        "method": "message/send",
        "params": {"message": {"role": "user", "parts": [{"kind": "text", "text": report}]}}
    }
    result = httpx.post(f"{A2A_NOTIFY_URL}/a2a", json=payload).json()
    task = result.get("result", {})
    print(f"📤 Task 상태: {task.get('status', {}).get('state')}")
    for artifact in task.get("artifacts", []):
        for part in artifact.get("parts", []):
            if part.get("kind") == "text":
                print(f"\n{part['text']}")
    return task

def hitl_checkpoint(report):
    banner("HITL 확장 포인트")
    print("""
현재 데모에서 HITL을 추가할 수 있는 3곳:

① [메일 필터 전] 사용자가 필터 조건을 동적으로 변경
   → LangGraph의 interrupt_before 노드 사용

② [요약 후] 긴급 판별 결과를 사용자가 승인/거부
   → DeepAgent에 human_approval 도구 추가

③ [알림 전송 전] 최종 알림 내용을 사용자가 확인
   → A2A Task 상태를 'input-required'로 설정
""")

if __name__ == "__main__":
    banner("🚀 메일 Agent End-to-End 데모")
    data = phase1_check_mail()
    report = phase2_generate_report(data)
    task = phase3_notify_via_a2a(report)
    hitl_checkpoint(report)
    banner("✅ 데모 완료")
    print("""
다음 단계:
  - 실제 IMAP/Gmail API 연동
  - Slack Webhook으로 알림 전송
  - LangGraph interrupt로 HITL 구현
  - 크론 스케줄러로 자동 폴링
""")
```

---

## 6.3 HITL — LangGraph interrupt 패턴

```python
"""HITL 확장: LangGraph interrupt를 사용한 사용자 승인 패턴"""
from langgraph.types import interrupt

def review_node(state):
    approval = interrupt({
        "question": "이 알림을 전송할까요?",
        "report_preview": state["report"][:200],
        "options": ["전송", "수정", "취소"]
    })
    if approval == "전송":
        return {"action": "send", "approved": True}
    elif approval == "수정":
        return {"action": "edit", "approved": False}
    else:
        return {"action": "cancel", "approved": False}
```

---

## 6.4 전체 커리큘럼 회고

```
Module 6: 통합 Demo
  ↑
Module 5: A2A (Agent ↔ Agent 통신)
  ↑
Module 4: Skills & MCP (Agent ↔ Tool 표준화)
  ↑
Module 3: DeepAgents (Harness — 배터리 포함 Agent)
  ↑
Module 2: LangChain/LangGraph (Framework + Runtime)
  ↑
Module 1: Agent 패러다임 (이론 기초)
```

### 핵심 Take-away

| 개념 | 한 줄 요약 |
|------|----------|
| Agent | LLM + Tools + Loop |
| ReAct | Think → Act → Observe 반복 |
| LangChain | 추상화 블록 (Framework) |
| LangGraph | 상태 머신 실행 엔진 (Runtime) |
| DeepAgents | 즉시 동작하는 Agent 키트 (Harness) |
| Skill | 관련 도구 + 프롬프트 + 후처리를 묶은 역량 패키지 |
| MCP | Agent ↔ Tool 표준 프로토콜 |
| A2A | Agent ↔ Agent 표준 프로토콜 |
| HITL | Agent 실행 중 사람이 개입하는 포인트 |

---

## 6.5 최종 체크포인트

> - [ ] Module 1: Framework/Runtime/Harness 차이를 설명할 수 있다
> - [ ] Module 2: LangGraph Agent가 상태를 유지하며 동작한다
> - [ ] Module 3: DeepAgents로 동일 기능을 더 적은 코드로 구현했다
> - [ ] Module 4: Skill이 MCP 결과를 정규화하여 전달하는 흐름을 이해한다
> - [ ] Module 5: A2A로 두 Agent가 Task/Artifact를 주고받는다
> - [ ] Module 6: E2E 파이프라인이 동작하고, HITL 확장 포인트를 식별했다

---

## 참고 자료

- [LangChain Docs](https://docs.langchain.com)
- [LangGraph Docs](https://docs.langchain.com/oss/python/langgraph)
- [DeepAgents Docs](https://docs.langchain.com/oss/python/deepagents/overview)
- [DeepAgents GitHub](https://github.com/langchain-ai/deepagents)
- [A2A Protocol](https://a2a-protocol.org/latest/)
- [A2A Python SDK](https://github.com/a2aproject/a2a-python)
- [MCP Specification](https://modelcontextprotocol.io)
- [Agent Frameworks, Runtimes, and Harnesses](https://blog.langchain.com/agent-frameworks-runtimes-and-harnesses-oh-my/)
- [Doubling Down on DeepAgents](https://blog.langchain.com/doubling-down-on-deepagents/)
- [Google A2A 발표 블로그](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
