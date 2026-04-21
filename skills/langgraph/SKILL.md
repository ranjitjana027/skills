---
name: langgraph
description: Build stateful AI applications with LangGraph in Python. Trigger when users ask about StateGraph, MessagesState, nodes, edges, ToolNode, checkpointers, streaming, subgraphs, Graph API, Functional API, deployment, or LangGraph runtime errors.
license: Complete terms in LICENSE.txt
---

# LangGraph (Python)

Use this skill to design, implement, and debug LangGraph applications using official LangGraph patterns.

## Core Workflow

1. Choose API style: Graph API or Functional API.
2. Define typed graph state.
3. Implement nodes that return partial state updates.
4. Add static and conditional edges.
5. Add tools and routing.
6. Add persistence/checkpointing for resumability.
7. Add tests, streaming, and observability.

## Minimal Graph Example

```python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    input: str
    output: str


def process_node(state: State) -> dict:
    return {"output": state["input"].strip()}

builder = StateGraph(State)
builder.add_node("process", process_node)
builder.add_edge(START, "process")
builder.add_edge("process", END)
graph = builder.compile()
```

## MessagesState + ToolNode Pattern

```python
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.prebuilt import ToolNode
from langchain_core.messages import SystemMessage

class State(MessagesState):
    route: str


def agent_node(state: State) -> dict:
    response = llm.bind_tools(tools).invoke(
        [SystemMessage(content="You are a helpful assistant")] + state["messages"]
    )
    return {"messages": [response]}


def should_continue(state: State):
    last = state["messages"][-1]
    return "tools" if getattr(last, "tool_calls", None) else END

builder = StateGraph(State)
builder.add_node("agent", agent_node)
builder.add_node("tools", ToolNode(tools))
builder.add_edge(START, "agent")
builder.add_conditional_edges("agent", should_continue, ["tools", END])
builder.add_edge("tools", "agent")
graph = builder.compile()
```

## Persistence

```python
from langgraph.checkpoint.memory import MemorySaver

graph = builder.compile(checkpointer=MemorySaver())
result = graph.invoke(
    {"messages": [...]},
    config={"configurable": {"thread_id": "thread-1"}},
)
```

## Streaming

```python
for chunk in graph.stream(
    {"messages": [...]},
    config={"configurable": {"thread_id": "thread-1"}},
    stream_mode="values",
):
    print(chunk)
```

## Common Errors To Check First

- `GRAPH_RECURSION_LIMIT`: Ensure termination conditions exist.
- `INVALID_GRAPH_NODE_RETURN_VALUE`: Return valid partial state dictionaries.
- `MISSING_CHECKPOINTER`: Configure checkpointer when persistence is required.
- `INVALID_CONCURRENT_GRAPH_UPDATE`: Avoid conflicting concurrent state writes.
- `INVALID_CHAT_HISTORY`: Ensure message sequence is valid for your model/tool calls.

## Best Practices

- Keep state schema minimal and explicit.
- Keep node logic deterministic where possible.
- Add one routing branch at a time and test each path.
- Use subgraphs for modular composition.
- Add observability and tests before deployment.

## Official Docs

- Overview: https://docs.langchain.com/oss/python/langgraph/overview
- Install: https://docs.langchain.com/oss/python/langgraph/install
- Quickstart: https://docs.langchain.com/oss/python/langgraph/quickstart
- Graph API: https://docs.langchain.com/oss/python/langgraph/graph-api
- Functional API: https://docs.langchain.com/oss/python/langgraph/functional-api
- Common errors: https://docs.langchain.com/oss/python/langgraph/common-errors
