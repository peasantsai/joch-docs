# LangGraph Integration

[**LangGraph**](https://langchain-ai.github.io/langgraph/) is LangChain's graph-based agent runtime: stateful workflows expressed as a directed graph of nodes, edges, and a typed state store. Joch attaches to it through the `langgraph` framework adapter.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `StateGraph` | The typed graph builder. Defines nodes, edges, and a `TypedDict`-style state shape. |
| `Node` (callable) | A function that receives state and returns state delta. |
| `Edge` / `add_conditional_edges` | Static and dynamic transitions between nodes. |
| `Tool` (LangChain) | Tool used inside a node. |
| `Checkpointer` (e.g., `MemorySaver`, `PostgresSaver`, `RedisSaver`) | Persistence of graph state per thread. |
| `interrupt_before` / `interrupt_after` | Human-in-the-loop suspension points. |
| `compile()`, `invoke()`, `stream()`, `astream()` | Entrypoints. |
| `Command` | Mutate state and route from inside a node. |
| MCP via tool nodes | LangChain MCP adapters in nodes. |
| Streaming | Full event streaming (`updates`, `values`, `messages`). |

## What Joch wraps and what it does not

```text
                ┌──────────────── your code ───────────────┐
                │   from langgraph.graph import StateGraph │
                │   g = StateGraph(MyState)                │
                │   g.add_node("agent", agent_fn)          │
                │   g.add_node("tools", tool_fn)           │
                │   g.add_edge("agent", "tools")           │
                │   app = g.compile(checkpointer=...)      │
                │   async for ev in app.astream(...):      │
                │       ...                                │
                └──┬───────────────┬───────────────┬───────┘
                   │ tool nodes    │ checkpointer  │ interrupts
                   ▼               ▼               ▼
   ── Joch boundary ─────────────────────────────────────────────
                   ▼
         ┌──────────────────────────────────────────────┐
         │ Joch langgraph FrameworkAdapter              │
         │   wraps tool node calls with Tool Gateway    │
         │   routes model nodes through Model Router    │
         │   bridges checkpointer to StateCheckpoint    │
         │   maps interrupt_before/after to Approval    │
         │   forwards graph events to Joch Trace        │
         └──────────────────────────────────────────────┘
```

## Mapping table

| LangGraph | Joch |
|---|---|
| `StateGraph` | One [`Agent`](../specs/kubernetes/agent.md) record with `framework.adapterRef: langgraph` and `framework.workflow.shape=graph` |
| `Node` (model-calling) | Routed through the [Model Router](../architecture/model-router.md) |
| `Node` (tool-calling) | One [`Tool`](../specs/kubernetes/tool.md) record per tool; routed through the [Tool Gateway](../architecture/tool-gateway.md) |
| Conditional edges | Recorded as routing events in the trace |
| `Checkpointer` | Bridged to [`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md) |
| `interrupt_before`, `interrupt_after` | Bound to [`Approval`](../specs/kubernetes/approval.md) for non-blocking human-in-the-loop |
| MCP tool node | One [`MCPServer`](../specs/kubernetes/mcpserver.md) record; gateway proxies the call |
| `stream()`, `astream()` | Streamed end-to-end through the gateway boundary |

## The smallest install

### Before

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated, Sequence
from langchain_core.messages import BaseMessage
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import ToolNode

class State(TypedDict):
    messages: Annotated[Sequence[BaseMessage], "messages"]

llm = ChatOpenAI(model="gpt-5-thinking").bind_tools([search_zendesk])
agent_node = lambda s: {"messages": [llm.invoke(s["messages"])]}
tools_node = ToolNode([search_zendesk])

g = StateGraph(State)
g.add_node("agent", agent_node)
g.add_node("tools", tools_node)
g.set_entry_point("agent")
g.add_conditional_edges("agent", lambda s: "tools" if s["messages"][-1].tool_calls else END)
g.add_edge("tools", "agent")
app = g.compile(checkpointer=MemorySaver())
```

### After — Joch-governed

```python
# ... same StateGraph / nodes as before ...
from joch.adapters.langgraph import bind_runtime

app = bind_runtime(
    g.compile(checkpointer=MemorySaver()),
    joch_agent_ref="support-triage",
    server_url="http://joch-server:8080",
)
```

`bind_runtime(...)` injects:

- a wrapper around tool nodes that emits `steps/toolCallRequest` / `steps/toolCallResult` and applies `allow` / `deny` / `modify`,
- a wrapper around model-calling nodes that routes through the Joch model router,
- a checkpointer adapter that mirrors checkpoints into [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md),
- an `interrupt_before` / `interrupt_after` interceptor that creates `Approval` records,
- a streaming adapter that emits trace events for graph transitions and node outputs.

## What Joch leaves to the SDK

- The graph structure, edge logic, and routing.
- Native checkpointer storage backends.
- Streaming event semantics.
- LangChain tool definitions and inputs.

## Reference

- LangGraph docs: <https://langchain-ai.github.io/langgraph/>
- Examples: <https://github.com/langchain-ai/langgraph/tree/main/examples>
