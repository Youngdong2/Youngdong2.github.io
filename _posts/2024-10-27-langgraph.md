---
layout: single
title: "LangGraph 튜토리얼 완벽 정리"
categories: LLM
use_math: false
comments: true
toc: true
author_profile: false
---

이번 포스팅에서는 최근 관심이 높아진 LangGraph 튜토리얼에 대해 정리해보겠습니다. 이 포스팅은 [공식 튜토리얼 문서](https://langchain-ai.github.io/langgraph/tutorials/introduction/)를 참고했습니다.

목차는 다음과 같습니다.

1. LangGraph의 목적
2. Build a Basic Chatbot
3. Enhancing the Chatbot with Tools
4. Adding Memory to the Chatbot
5. Human-in-the-loop
6. Manually Updating the State
7. Customizing State
8. Time Travel

## LangGraph의 목적

LangGraph가 무엇이고 왜 쓰는 것일까요?

LangChain을 통해 LLM flow를 개발하면서 힘들었던 점은 이전 체인의 결과를 다음 체인으로 넘길 때, 생성된 결과의 format이 안맞는 등의 실패를 겪어보신 적이 대부분 있을겁니다. 이런 경우 보통 Output Parser를 통해 해결하게 되는데 이 방법은 모니터링을 할 때 아쉽다라는 느낌이 없지 않습니다.

LangGraph는 LangChain 라이브러리 내에서 이러한 문제를 해결하기 위한 라이브러리입니다. 이 라이브러리는 여러 LLM chain 또는 Agent를 구조화된 방식으로 정의, 조정 및 실행할 수 있는 프레임워크를 제공합니다.

그럼 이제 튜토리얼을 시작해보죠!

## Part 1: Build a Basic Chatbot

이 세션에서는 LangGraph를 사용해 사용자 메시지에 직접 응답하는 간단한 챗봇을 만들면서, LangGraph의 핵심 개념을 소개합니다. 이 섹션이 끝나면 기본적인 챗봇을 만들 수 있습니다.

### 1. StateGraph 생성

LangGraph에서는 **StateGraph** 객체가 챗봇의 구조를 "state machine"로 정의합니다. `StateGraph`에 노드(챗봇이 호출할 LLM 또는 함수)를 추가하고, 엣지를 통해 노드 간의 전환을 정의합니다.

```python
from typing import Annotated

from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages


class State(TypedDict):
    # Messages have the type "list". The `add_messages` function
    # in the annotation defines how this state key should be updated
    # (in this case, it appends messages to the list, rather than overwriting them)
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)
```

- `State`클래스는 `TypeDict`로 정의되어, 그래프의 상태가 됩니다.
- 여기에는 `messages`라는 단일 키가 있으며, `add_messages`함수로 메시지가 기존 상태에 추가되도록 설정합니다.
- `graph_builder`는 `StateGraph`로 초기화되어, `State`의 스키마와 상태 갱신 방식인 reducer 함수를 정의합니다.

> **Note**: 그래프를 정의할 때 첫 번째 작업은 **State**를 정의하는 것. State는 그래프의 스키마와 상태 업데이트 방식을 정의하는데, 여기서 `messages`키는 **add_messages** reducer 함수가 적용되어 새로운 메시지를 덮어쓰지 않고 기존 목록에 추가되도록 합니다. Annotated 문법을 사용하여 이를 지정합니다. 다른 키는 기본적으로 덮어쓰기 방식으로 저장됩니다.

이로써 그래프는 두 가지를 알게 됩니다.

1. 우리가 정의할 모든 노드는 현재 상태를 입력으로 받고, 상태를 업데이트하는 값을 반환
2. `messages`는 덮어쓰기가 아닌 **add_messages** 함수를 통해 기존 목록에 메시지를 추가

### 2. 챗봇 노드 추가

이제 "chatbot" 노드를 추가합니다. 노드는 작업 단위를 나타냅니다. 공식 튜토리얼에서는 **ChatAntropic** 모델을 사용해 응답을 생성하는 챗봇을 구현합니다.

```python
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-3-5-sonnet-20240620")


def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}


# 첫 번째 인자는 노드의 고유 이름
# 두 번째 인자는 노드가 호출될 때 실행되는 함수 또는 객체
graph_builder.add_node("chatbot", chatbot)
```

- chatbot 함수는 현재 State를 입력받고, 갱신된 `messages` 리스트를 반환합니다. 이 방식은 모든 LangGraph 노드 함수에서 기본적으로 적용됩니다.
- 다음으로, **add_node** 메서드를 사용해 챗봇 노드를 그래프에 추가합니다.

> **Note** : 여기에서 `chatbot` 노드 함수는 현재 상태를 입력으로 받고, `messages`키 아래에 갱신된 메시지 목록을 포함한 딕셔너리를 반환합니다. LangGraph는 `add_messages` 함수를 통해 LLM의 응답을 현재 상태에 있는 메시지 목록에 추가합니다.

### 3. 시작점과 종료점 설정

이제 챗봇의 시작점과 종료점을 설정합니다. 시작점은 그래프가 각 실행 시 작업을 시작할 위치를 지정하며, 종료점은 특정 노드가 실행되면 스래프가 종료될 수 있음을 알려줍니다.

```python
graph_builder.add_edge(START, "chatbot") # 시작점 설정
graph_builder.add_edge("chatbot", END) # 종료점 설정
```

### 4. 그래프 컴파일 및 실행

모든 설정이 완료되면, `compile()`을 호출하여 그래프를 컴파일합니다.
**CompiledGraph** 객체를 생성하여 상태 기반으로 동작을 수행하게 됩니다.

```python
graph = graph_builder.compile()
```

또한 `get_graph`메서드와 `draw_ascii` 또는 `draw_png` 와 같은 그리기 메서드를 사용해 그래프를 시각화할 수 있습니다.

```python
from IPython.display import Image, display

try:
    display(Image(graph.get_graph().draw_mermaid_png()))
except Exception:
    # 시각화에 추가 종속성이 필요
    pass
```

<p align="center">
  <img src="/images/langgraph/langgraph1.jpeg" height="200px" width="100px">
</p>

### 5. 챗봇 실행하기

이제 챗봇과 상호작용할 수 있는 간단한 대화 루프를 만듭니다. `stream_graph_updates`함수는 사용자 입력을 기반으로 챗봇의 응답을 출력합니다. 사용자는 "quit", "exit" 또는 "q"를 입력해 대화를 종료할 수 있습니다.

```python
def stream_graph_updates(user_input: str):
    for event in graph.stream({"messages": [("user", user_input)]}):
        for value in event.values():
            print("Assistant:", value["messages"][-1].content)


while True:
    try:
        user_input = input("User: ")
        if user_input.lower() in ["quit", "exit", "q"]:
            print("Goodbye!")
            break

        stream_graph_updates(user_input)
    except:
        # fallback if input() is not available
        user_input = "What do you know about LangGraph?"
        print("User: " + user_input)
        stream_graph_updates(user_input)
        break
```

```text
Assistant: LangGraph is a library designed to help build stateful multi-agent applications using language models. It provides tools for creating workflows and state machines to coordinate multiple AI agents or language model interactions. LangGraph is built on top of LangChain, leveraging its components while adding graph-based coordination capabilities. It's particularly useful for developing more complex, stateful AI applications that go beyond simple query-response interactions.
Goodbye!
```

## Part 2: Enhancing the Chatbot with Tools

이번에는 웹 검색 도구를 통합하여 답변을 개선하는 방법을 알아봅시다.

시작하기 전에 필요한 패키지를 설치하고 API 키를 설정해야 합니다. 튜토리얼에서는 Tavily 검색 엔진을 사용합니다.

```python
%%capture --no-stderr
%pip install -U tavily-python langchain_community

_set_env("TAVILY_API_KEY")
```

### 1. Tool 정의

Tavily 검색 도구를 정의하여, 챗봇이 'LangGraph의 노드가 무엇인가요?'와 같은 질문에 웹 검색을 통해 답변을 찾을 수 있도록 합니다.

```python
from langchain_community.tools.tavily_search import TavilySearchResults

tool = TavilySearchResults(max_results=2)
tools = [tool]
tool.invoke("What's a 'node' in LangGraph?")
```

이때, Tavily 검색 도구는 페이지 요약 정보를 반환하여 챗봇이 사용할 수 있습니다.

### 2. StateGraph 및 LLM 초기화

기본적인 StateGraph 생성 코드는 part1과 유사하지만, **bind_tools** 메서드를 사용하여 LLM이 JSON 형식으로 검색 엔진을 호출할 수 있도록 합니다.

```python
from typing import Annotated

from langchain_anthropic import ChatAnthropic
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages


class State(TypedDict):
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)


llm = ChatAnthropic(model="claude-3-5-sonnet-20240620")
# Modification: tell the LLM which tools it can call
llm_with_tools = llm.bind_tools(tools)


def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}


graph_builder.add_node("chatbot", chatbot)
```

이렇게 하면 LLM이 적절한 JSON 형식으로 도구를 호출할 수 있습니다.

### 3. Tool 실행 노드 추가

Tool이 호출될 경우 이를 실행하는 **BasicToolNode** 클래스를 정의합니다. 이 노드는 최근 메시지에서 `tool_calls`가 포함된 경우 tool을 호출합니다.

```python
import json

from langchain_core.messages import ToolMessage


class BasicToolNode:
    """AIMesage의 마지막 메시지에 포함된 도구 요청을 실행하는 노드."""

    def __init__(self, tools: list) -> None:
        self.tools_by_name = {tool.name: tool for tool in tools}

    def __call__(self, inputs: dict):
        if messages := inputs.get("messages", []):
            message = messages[-1]
        else:
            raise ValueError("No message found in input")
        outputs = []
        for tool_call in message.tool_calls:
            tool_result = self.tools_by_name[tool_call["name"]].invoke(
                tool_call["args"]
            )
            outputs.append(
                ToolMessage(
                    content=json.dumps(tool_result),
                    name=tool_call["name"],
                    tool_call_id=tool_call["id"],
                )
            )
        return {"messages": outputs}


tool_node = BasicToolNode(tools=[tool])
graph_builder.add_node("tools", tool_node)
```

위의 `BasicToolNode`는 메시지에서 `tool_calls`가 있는지확인하여 도구를 호출하고 결과를 반환합니다.

### 4. 조건부 엣지 설정

조건부 엣지는 특정 조건에 따라 노드 간 흐름을 제어합니다. 여기서는 **route_tools** 함수로 `tool_calls`가 있는지 확인하고, 있느면 `tools`로, 없으면 `END`로 이동합니다.

```python
from typing import Literal


def route_tools(
    state: State,
):
    """
    Use in the conditional_edge to route to the ToolNode if the last message
    has tool calls. Otherwise, route to the end.
    """
    if isinstance(state, list):
        ai_message = state[-1]
    elif messages := state.get("messages", []):
        ai_message = messages[-1]
    else:
        raise ValueError(f"No messages found in input state to tool_edge: {state}")
    if hasattr(ai_message, "tool_calls") and len(ai_message.tool_calls) > 0:
        return "tools"
    return END

graph_builder.add_conditional_edges(
    "chatbot",
    route_tools,
    {"tools": "tools", END: END},
)
# Any time a tool is called, we return to the chatbot to decide the next step
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge(START, "chatbot")
graph = graph_builder.compile()
```

이제 `chatbot` 노드가 실행될 때마다 `tool_calls`가 있으면 `tools`로, 그렇지 않으면 `END`로 이동하도록 합니다.

#### 5. 그래프 시각화 및 실행

```python

from IPython.display import Image, display

try:
    display(Image(graph.get_graph().draw_mermaid_png()))
except Exception:
    # This requires some extra dependencies and is optional
    pass
```

<p align="center">
  <img src="/images/langgraph/langgraph2.jpeg" height="200px" width="150px">
</p>

이제 챗봇은 학습 데이터 외부의 질문에 답변할 수 있으며, 사용자 입력에 따라 Tavily 검색 엔진을 통해 정보를 찾고 제공합니다.

```python
while True:
    try:
        user_input = input("User: ")
        if user_input.lower() in ["quit", "exit", "q"]:
            print("Goodbye!")
            break

        stream_graph_updates(user_input)
    except:
        # fallback if input() is not available
        user_input = "What do you know about LangGraph?"
        print("User: " + user_input)
        stream_graph_updates(user_input)
        break
```

예시 응답:

```css
Assistant: [{"url": "https://www.langchain.com/langgraph", "content": "LangGraph는 복잡한 AI 작업을 수행하는 데 도움이 되는 라이브러리입니다..."}]
```

## Part3: Adding Memory to the Chatbot

Part2까지의 과정을 통해 챗봇이 tool을 활용하여 질문에 답할 수 있지만, 이전 대화 맥락을 기억하지는 못합니다.
이는 멀티턴 대화의 연속성을 유지하는 데 한계가 있습니다. **LangGraph**는 지속적인 체크포인트 시스템을 통해 이 문제를 해결합니다.

### LangGraph의 Checkpointing 시스템

LangGraph에서 체크포인터와 `thread_id`를 지정해 그래프를 컴파일하면, 그래프는 각 단계 후 상태를 자동으로 저장합니다. 동일한 `thred_id`로 다시 호출하면 저장된 상태가 로드되어 챗봇이 이전 대화에서 이어갈 수 있습니다.

> **Note**: 
> Checkpoing은 단순한 메모리 이상의 기능을 제공합니다. 이는 에러 복구, human-in-the-loop 워크플로우, time travel interactions 등을 가능하게 합니다. 먼저 멀티턴 대화를 위한 체크포인트를 추가해 보겠습니다.

### 1. 메모리 saver 생성

먼저 **MemorySaver** 체크포인터를 생성합니다.

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
```

이 체크포인터는 메모리에 데이터를 저장합니다. 실제 운영 환경에서는 SQLite나 PostgreSQL과 같은 DB에 연결할 수 있는 **SqliteSaver**나 **PostgresSaver**를 사용할 수 잇습니다.

### 2. 그래프 정의 및 컴파일

기존의 **BasicToolNode** 대신 LangGraph의 **ToolNode**와 **tools_condition**을 사용하여 더 효율적은 API 설행을 구성합니다. 나머지 코드 구조는 part2와 유사합니다.

```python
from typing import Annotated

from langchain_anthropic import ChatAnthropic
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.messages import BaseMessage
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition


class State(TypedDict):
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)


tool = TavilySearchResults(max_results=2)
tools = [tool]
llm = ChatAnthropic(model="claude-3-5-sonnet-20240620")
llm_with_tools = llm.bind_tools(tools)


def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}


graph_builder.add_node("chatbot", chatbot)

tool_node = ToolNode(tools=[tool])
graph_builder.add_node("tools", tool_node)

graph_builder.add_conditional_edges(
    "chatbot",
    tools_condition,
)
# Any time a tool is called, we return to the chatbot to decide the next step
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge(START, "chatbot")
```

이제 체크포인터와 함께 그래프를 컴파일합니다.

```python
graph = graph_builder.compile(checkpointer=memory)
```

### 3. 챗봇과 상호작용

각 대화의 **thread_id**를 통해 챗봇이 대화 맥락을 기억하게 됩니다.

```python
config = {"configurable": {"thread_id": "1"}}
```

다음과 같이 챗봇을 호출해봅시다.

```python
user_input = "Hi there! My name is Will."

# The config is the **second positional argument** to stream() or invoke()!
events = graph.stream(
    {"messages": [("user", user_input)]}, config, stream_mode="values"
)
for event in events:
    event["messages"][-1].pretty_print()
```

응답 예:

```text
================================ Human Message =================================
Hi there! My name is Will.
================================= Ai Message ===================================
Hello Will! It's nice to meet you. How can I assist you today?

```

이제 이름을 기억하는지 확인합니다.

```python
user_input = "Remember my name?"

# The config is the **second positional argument** to stream() or invoke()!
events = graph.stream(
    {"messages": [("user", user_input)]}, config, stream_mode="values"
)
for event in events:
    event["messages"][-1].pretty_print()
```

```text
================================ Human Message =================================
Remember my name?
================================= Ai Message ===================================
Of course, I remember your name, Will. Is there anything else you'd like to talk about?
```

> **Note**:
> 메모리는 LangGraph의 체크포인터로 처리되므로, 외부 리스트나 추가 메모리가 필요하지 않습니다.

새로운 `thread_id`로 호출하면 챗봇은 초기 상태로 돌아갑니다.

```python
# The only difference is we change the `thread_id` here to "2" instead of "1"
events = graph.stream(
    {"messages": [("user", user_input)]},
    {"configurable": {"thread_id": "2"}},
    stream_mode="values",
)
for event in events:
    event["messages"][-1].pretty_print()
```

```text
================================ Human Message =================================
Remember my name?
================================= Ai Message ===================================
I apologize, but I don't have any previous context or memory of your name. Could you please tell me your name?
```

단순히 `thread_id`만 변경했음에도 챗봇은 새로운 대화로 인식합니다.

### 상태 스냅샷 검사하기

**get_state** 메서드를 통해 현재 상태를 확인할 수 있습니다.

```python
snapshot = graph.get_state(config)
snapshot
```

```text
StateSnapshot(values={'messages': [HumanMessage(content='Hi there! My name is Will.', additional_kwargs={}, response_metadata={}, id='8c1ca919-c553-4ebf-95d4-b59a2d61e078'), AIMessage(content="Hello Will! It's nice to meet you. How can I assist you today? Is there anything specific you'd like to know or discuss?", additional_kwargs={}, response_metadata={'id': 'msg_01WTQebPhNwmMrmmWojJ9KXJ', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 405, 'output_tokens': 32}}, id='run-58587b77-8c82-41e6-8a90-d62c444a261d-0', usage_metadata={'input_tokens': 405, 'output_tokens': 32, 'total_tokens': 437}), HumanMessage(content='Remember my name?', additional_kwargs={}, response_metadata={}, id='daba7df6-ad75-4d6b-8057-745881cea1ca'), AIMessage(content="Of course, I remember your name, Will. I always try to pay attention to important details that users share with me. Is there anything else you'd like to talk about or any questions you have? I'm here to help with a wide range of topics or tasks.", additional_kwargs={}, response_metadata={'id': 'msg_01E41KitY74HpENRgXx94vag', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 444, 'output_tokens': 58}}, id='run-ffeaae5c-4d2d-4ddb-bd59-5d5cbf2a5af8-0', usage_metadata={'input_tokens': 444, 'output_tokens': 58, 'total_tokens': 502})]}, next=(), config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef7d06e-93e0-6acc-8004-f2ac846575d2'}}, metadata={'source': 'loop', 'writes': {'chatbot': {'messages': [AIMessage(content="Of course, I remember your name, Will. I always try to pay attention to important details that users share with me. Is there anything else you'd like to talk about or any questions you have? I'm here to help with a wide range of topics or tasks.", additional_kwargs={}, response_metadata={'id': 'msg_01E41KitY74HpENRgXx94vag', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 444, 'output_tokens': 58}}, id='run-ffeaae5c-4d2d-4ddb-bd59-5d5cbf2a5af8-0', usage_metadata={'input_tokens': 444, 'output_tokens': 58, 'total_tokens': 502})]}}, 'step': 4, 'parents': {}}, created_at='2024-09-27T19:30:10.820758+00:00', parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef7d06e-859f-6206-8003-e1bd3c264b8f'}}, tasks=())
```

이 스냅샷에는 현재 상태, 해당 설정, 다음 처리 노드가 포함됩니다. 이 예시에서는 그래프가 `END`상태에 도달했으므로 `next`는 비어있습니다.

## Part4: Human-in-the-loop

Agent가 모든 작업을 신뢰할 수 있는 방식으로 수행하지 못할 때는 인간의 검토나 입력이 필요할 수 있습니다.
LangGraph는 **interrupt_before** 기능을 통해 특정 노드에서 작업을 일시중지하고, 인간이 검토하거나 승인한 후에만 작업을 진행할 수 있게 합니다.

이번 파트에서는 **tool** 노드 실행 전마다 작업을 중단하도록 설정하여 인간 검토가 필요할 때 개입할 수 있는 구조를 만들어보겠습니다.

### 1. 기존 코드 불러오기

이 섹션의 코드는 part 3의 설정을 기반으로 합니다.

```python
from typing import Annotated

from langchain_anthropic import ChatAnthropic
from langchain_community.tools.tavily_search import TavilySearchResults
from typing_extensions import TypedDict

from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition

memory = MemorySaver()


class State(TypedDict):
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)

tool = TavilySearchResults(max_results=2)
tools = [tool]
llm = ChatAnthropic(model="claude-3-5-sonnet-20240620")
llm_with_tools = llm.bind_tools(tools)


def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}


graph_builder.add_node("chatbot", chatbot)

tool_node = ToolNode(tools=[tool])
graph_builder.add_node("tools", tool_node)

graph_builder.add_conditional_edges(
    "chatbot",
    tools_condition,
)
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge(START, "chatbot")
```

### 2. 그래프 컴파일 및 중단 지점 설정

**interrupt_before** 인수를 통해 `tools`노드 실행 전 작업을 중단하도록 지정합니다. 이를 통해 인간 검토 후 작업을 재개할 수 있습니다.

```python
graph = graph_builder.compile(
    checkpointer=memory,
    # This is new!
    interrupt_before=["tools"],
)
```

### 3. 챗봇과 상호작용 및 검토

아래 코드는 챗봇에 입력을 전달하고, 중단 지점에서 상태를 확인하는 방법을 보여줍니다.

```python
user_input = "I'm learning LangGraph. Could you do some research on it for me?"
config = {"configurable": {"thread_id": "1"}}

events = graph.stream(
    {"messages": [("user", user_input)]}, config, stream_mode="values"
)
for event in events:
    if "messages" in event:
        event["messages"][-1].pretty_print()
```

출력 예:

```test
================================[1m Human Message [0m=================================

I'm learning LangGraph. Could you do some research on it for me?
==================================[1m Ai Message [0m==================================

[{'text': "Certainly! I'd be happy to research LangGraph for you. To get the most up-to-date and comprehensive information, I'll use the Tavily search engine to look this up. Let me do that for you now.", 'type': 'text'}, {'id': 'toolu_01R4ZFcb5hohpiVZwr88Bxhc', 'input': {'query': 'LangGraph framework for building language model applications'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}]
Tool Calls:
  tavily_search_results_json (toolu_01R4ZFcb5hohpiVZwr88Bxhc)
 Call ID: toolu_01R4ZFcb5hohpiVZwr88Bxhc
  Args:
    query: LangGraph framework for building language model applications
```

이 시점에서 **graph.get_state(config)**로 상태를 확인하여 `tools`노드에 중단된 상태를 확인할 수 있습니다.

```python
snapshot = graph.get_state(config)
snapshot.next
```

```text
('tools',)
```

**next** 값이 `tools`로 설정되어 있으면, 중단된 상태에서 검토를 수행한 수 작업을 계속할 수 있습니다.

### 4. 작업 재개

검토 후 `None`을 전달해 그래프가 중단된 지점에서 이어서 실행되도록 합니다.

```python
events = graph.stream(None, config, stream_mode="values")
for event in events:
    if "messages" in event:
        event["messages"][-1].pretty_print()
```

챗봇은 이전 중단 지점에서 이어받아 결과를 제공합니다.

이제 **interrupt_before**기능을 활용해 인간의 개입을 요구하는 단계에서 작업을 중단할 수 있습니다. 이를 통해 agent의 신뢰성을 높이고, 필요한 경우 인간의 검토 후 작업을 이어갈 수 있게 되었습니다.

LangGraph의 체크포인터가 이미 포함되어 있으므로, 작업을 무기한 중단하고 언제든지 이어서 작업을 수행할 수 있습니다.

여기까지 LangGraph를 활용한 챗봇 구축의 흐름을 알아보았습니다. 끝까지 정리를 하면 포스팅이 너무 길어질 것 같아 나머지 파트는 다음 포스팅에서 정리해보도록 하겠습니다.

긴 글 읽어주셔서 감사합니다!