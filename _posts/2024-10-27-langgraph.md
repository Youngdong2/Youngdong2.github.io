---
layout: single
title: "LangGraph 튜토리얼 완벽 정리"
categories: LLM
use_math: false
comments: true
toc: true
author_profile: false
---

이번 포스팅에서는 최근 관심이 높아진 LangGraph 튜토리얼에 대해 정리해보겠습니다. 이 포스팅은 공식 튜토리얼 문서를 참고했습니다.

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