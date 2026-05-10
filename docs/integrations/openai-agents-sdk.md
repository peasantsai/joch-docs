# OpenAI Agents SDK Integration

The [**OpenAI Agents SDK**](https://openai.github.io/openai-agents-python/) is OpenAI's open-source production SDK for code-first agents. Joch attaches to it through the `openai-agents-sdk` framework adapter and governs the boundary between the SDK and the rest of the world.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `Agent` | An LLM equipped with `instructions`, `tools`, `handoffs`, `guardrails`, optional `output_type`, and a `model`. |
| `Runner` | Drives the agent loop. `Runner.run()`, `Runner.run_sync()`, `Runner.run_streamed()`. |
| `RunContextWrapper[Context]` | Per-run context object available to tools, guardrails, and lifecycle hooks. |
| `function_tool` | Decorator that turns a Python function into a tool with a Pydantic-derived JSON schema. |
| `MCPServer` | First-class hosted-MCP integration; exposes a server's tools to an agent. |
| Handoffs | Agents transferring control to other agents (`Agent.handoffs=[...]` or `handoff(...)`). |
| Guardrails | `input_guardrails` and `output_guardrails` validators. `tripwire_triggered` raises `InputGuardrailTripwireTriggered` / `OutputGuardrailTripwireTriggered`. |
| `Session` | Persistent conversation memory; ships with `SQLAlchemySession`, `SQLiteSession`, `RedisSession`, `MongoDBSession`, `DaprSession`, `EncryptedSession`, plus your own. |
| `Trace`, `Span` | Built-in tracing. Span types include `agent_span`, `generation_span`, `function_span`, `handoff_span`, `guardrail_span`, `custom_span`. |
| `AgentHooks`, `RunHooks` | Lifecycle hooks: `on_start`, `on_end`, `on_handoff`, `on_tool_start`, `on_tool_end`, `on_llm_start`, `on_llm_end`. |
| `RunConfig` | Per-run configuration: model, model provider, tracing settings, workflow name, group id. |
| Realtime + Voice | `RealtimeAgent`, `RealtimeRunner`, `RealtimeSession`; voice pipeline with STT/TTS providers. |

## What Joch wraps and what it does not

```text
                ┌───────────────── your code ─────────────────┐
                │   from agents import Agent, Runner, ...     │
                │   triage = Agent(...)                       │
                │   await Runner.run(triage, input="...")     │
                └──┬───────────────┬──────────────────────────┘
                   │ tools         │ handoffs / guardrails
                   ▼               ▼
         function_tool    Agent.handoffs    AgentHooks / RunHooks
                   │
   ── Joch boundary ─────────────────────────────────────────────
                   ▼
         ┌──────────────────────────────────────────────┐
         │ Joch openai-agents-sdk FrameworkAdapter      │
         │   binds tools to Tool Gateway                │
         │   wraps MCPServer with MCP Gateway           │
         │   redirects model client to Model Router     │
         │   bridges Session to Conversation            │
         │   forwards SDK trace → Joch Trace (OTLP)     │
         │   surfaces deny/modify to your callers       │
         │   refreshes AgBOM on apply                   │
         └──────────────────────────────────────────────┘
```

Joch does **not** replace `Agent`, `Runner`, the loop, the planner, structured outputs, streaming, or guardrails. Your agent code stays exactly as it was authored.

## Mapping table

| OpenAI Agents SDK | Joch |
|---|---|
| `Agent(name, instructions, tools, model, ...)` | One [`Agent`](../specs/kubernetes/agent.md) record with `framework.adapterRef: openai-agents-sdk` |
| `Agent.tools` (function_tool) | One [`Tool`](../specs/kubernetes/tool.md) record per tool; routed through the [Tool Gateway](../architecture/tool-gateway.md) |
| `MCPServer(...)` (hosted MCP) | One [`MCPServer`](../specs/kubernetes/mcpserver.md) record; routed through the [MCP Gateway](../architecture/mcp-gateway.md) |
| `Agent.handoffs` | [`Handoff`](../specs/kubernetes/handoff.md) records |
| `Agent.input_guardrails`, `Agent.output_guardrails` | Composed with [`Policy`](../specs/kubernetes/policy.md) at the gateway boundary |
| `Session` (any backend) | Bridged to [`Conversation`](../specs/kubernetes/conversation.md) + canonical message log |
| `Trace`, span types | Mirrored into [`Trace`](../specs/kubernetes/trace.md) + OTLP / OCSF export |
| `AgentHooks` / `RunHooks` | Continue to fire in-process; Joch additionally records `HookDecision` events |
| `RunConfig.model`, `RunConfig.model_provider` | [`ModelRoute`](../specs/kubernetes/model-route.md) + [Model Router](../architecture/model-router.md) |
| `Runner.run_streamed()` | Streamed end-to-end through the model router |
| `Realtime*` | Realtime hook surface plus dedicated `realtime` adapter binding |

## The smallest install

### Before — your existing code

```python
from agents import Agent, Runner, function_tool

@function_tool
async def search_zendesk(query: str) -> list[dict]:
    ...

triage = Agent(
    name="support-triage",
    instructions="Triage tickets and draft replies.",
    tools=[search_zendesk],
    model="gpt-5-thinking",
)

result = await Runner.run(triage, input="Refund for order 12345")
print(result.final_output)
```

### After — Joch-governed, identical agent code

```python
from agents import Agent, Runner, function_tool
from joch.adapters.openai_agents_sdk import bind_runtime  # adapter package

@function_tool
async def search_zendesk(query: str) -> list[dict]:
    ...

triage = Agent(
    name="support-triage",
    instructions="Triage tickets and draft replies.",
    tools=[search_zendesk],
    model="gpt-5-thinking",
)

bind_runtime(
    agent=triage,
    joch_agent_ref="support-triage",        # links to the Agent record in Joch
    server_url="http://joch-server:8080",    # control plane
)

result = await Runner.run(triage, input="Refund for order 12345")
print(result.final_output)
```

`bind_runtime` is the only change. It:

- registers tool functions with the Joch tool gateway,
- swaps the model client for one that routes through the Joch model router,
- captures session events and writes a canonical `Conversation` log,
- forwards the SDK's `Trace` / `Span` to Joch trace,
- surfaces `deny` / `modify` Guardian decisions as native SDK exceptions (`AgentExecutionDenied`, `AgentToolModified`).

## Tools through the gateway

Function tools (`@function_tool`) and hosted tools alike are intercepted at the call boundary:

```text
Runner.run() ──▶ Agent decides to call tool
              ──▶ adapter sends steps/toolCallRequest to Policy Engine
              ──▶ allow / deny / modify
              ──▶ executes function (in-process) or MCP tool (via gateway)
              ──▶ adapter sends steps/toolCallResult
              ──▶ allow / deny / modify
              ──▶ Runner observes result
```

A `deny` raises a Joch-mapped exception inside the SDK; a `modify` rewrites the tool inputs (or the result) before the agent sees it. See [`ToolCall`](../specs/kubernetes/toolcall.md).

## MCP

The SDK supports hosted MCP via `MCPServer(...)`. The Joch adapter swaps the underlying transport so every `tools/call` flows through the [Joch MCP Gateway](../architecture/mcp-gateway.md):

```python
from agents.mcp import MCPServerStdio
from joch.adapters.openai_agents_sdk.mcp import wrap_server

raw = MCPServerStdio(params={"command": "npx", "args": ["@github/mcp-server"]})
mcp = wrap_server(raw, joch_mcpserver_ref="github")  # routes through MCP gateway

triage = Agent(
    name="support-triage",
    instructions="...",
    tools=[],
    mcp_servers=[mcp],
)
```

The MCP server must already exist as an [`MCPServer`](../specs/kubernetes/mcpserver.md) record in Joch (registered, pinned, trust-scored).

## Models

The SDK's model is replaced by a Joch-routed model client. `RunConfig.model` and `RunConfig.model_provider` are honored; on top of them, the [`ModelRoute`](../specs/kubernetes/model-route.md) configured in the `Agent` record drives capability matching, fallback, and budget enforcement.

## Sessions and state

The SDK ships several `Session` implementations. The Joch adapter persists a parallel canonical event log into [`Conversation`](../specs/kubernetes/conversation.md), so:

- the SDK's session keeps working in-process,
- Joch can replay the conversation against a different provider via [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md),
- the same conversation can survive an agent migration to another framework.

## Hooks and tracing

`AgentHooks` and `RunHooks` continue to run in-process. Joch adds parallel `HookDecision` events at every gateway boundary. The SDK's native `Trace` and span types are mirrored into Joch:

| SDK span | Joch trace event |
|---|---|
| `agent_span` | `AgentRun` (root) |
| `generation_span` | `ModelCallStarted` / `ModelCallCompleted` |
| `function_span` | `ToolCallRequested` / `ToolCallCompleted` |
| `handoff_span` | `Handoff` |
| `guardrail_span` | `HookDecision` (input/output guardrails) |
| `custom_span` | passthrough as a custom span |

OpenTelemetry semantic conventions are applied via the [OpenTelemetry mapping](../aos/opentelemetry-mapping.md).

## Approvals

When a Guardian Agent returns `deny` or `modify` requiring human approval, the adapter:

1. pauses the tool call (a non-blocking suspend at the Runner level),
2. creates an [`Approval`](../specs/kubernetes/approval.md) record,
3. routes notifications via the configured channels,
4. resumes (or denies) on decision.

The agent code does not need to know about approvals; it only sees a normal tool result, a denial exception, or a modified result.

## AgBOM

The agent's [`AgBOM`](../specs/kubernetes/agbom.md) lists:

- `joch.framework=openai-agents-sdk`, `joch.framework.version=<sdk-version>`,
- every `function_tool`, MCP server, and hosted tool,
- every model selectable by the active `ModelRoute`,
- every memory and RAG binding,
- every policy applied by the policy engine,
- the framework adapter package version.

Refresh triggers fire on any of those changes.

## Migration matrix

| Existing | Joch step |
|---|---|
| Agent code in your repo | `joch discover --framework openai-agents-sdk --path ./agents` produces stub records |
| `function_tool` definitions | Auto-registered as `Tool` records during discovery |
| `MCPServer(...)` usage | Replace with `wrap_server(...)` in code; create matching `MCPServer` records |
| `Session` (Sqlite / Redis / etc.) | Keep your `Session`; the adapter writes the canonical log in parallel |
| `RunHooks` / `AgentHooks` | Keep them; Joch adds `HookDecision` records at the gateway |
| Model selection | Replace `model="..."` with a `ModelRoute` in your `Agent` record |

## What Joch leaves to the SDK

- The agent loop semantics — planning, decision-making, structured output validation.
- Function tool argument parsing and Pydantic schema generation.
- `output_type` and structured-output coercion.
- Voice / Realtime pipelines (Joch wraps the model and tool calls but does not re-implement the realtime transport).
- Sandbox agent flow when used with `SandboxAgent` / `SandboxClient`.

## Reference

- OpenAI Agents SDK guide: <https://developers.openai.com/api/docs/guides/agents>
- Python reference: <https://openai.github.io/openai-agents-python/>
- TypeScript reference: <https://github.com/openai/openai-agents-typescript>
