# Google ADK Integration

[**Google ADK**](https://adk.dev/) (Agent Development Kit) is the open-source agent framework behind the Gemini Enterprise Agent Platform: agents, workflows, tools, sessions, memory, plugins, callbacks, evaluation, and deployment to Vertex AI Agent Engine, Cloud Run, or GKE. Joch attaches to it through the `google-adk` framework adapter.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `LlmAgent(name, model, instruction, tools, …)` | The primary LLM-driven agent. |
| `BaseAgent` | Foundation class for custom agents. |
| `SequentialAgent`, `ParallelAgent`, `LoopAgent` | Workflow agents — explicit graph-style orchestration. |
| `FunctionTool` | Wraps a Python / Java / Go callable. |
| `AgentTool` | Calls another agent as a tool. |
| `LongRunningFunctionTool` | Async / long-running tool with progress events. |
| `OpenAPIToolset` | REST integration via OpenAPI spec. |
| `MCPToolset` | Connect MCP servers (`StdioConnectionParams`, `StreamableHTTPConnectionParams`). |
| `GoogleSearchTool`, `BuiltInCodeExecutor` | Built-in capabilities. |
| `Session` / `SessionService` | `InMemorySessionService`, `DatabaseSessionService`, `VertexAiSessionService`. |
| `BaseMemoryService` | `InMemoryMemoryService`, `VertexAiMemoryService`. |
| `Runner` | Executes agents; produces `Event` stream. |
| `Event` | Append-only run events surfaced to the caller. |
| `BaseArtifactService` | Stores artifacts produced during a run. |
| Plugins | Extension points for custom auth, persistence, telemetry. |
| Callbacks | `before_agent_callback`, `after_agent_callback`, `before_model_callback`, `after_model_callback`, `before_tool_callback`, `after_tool_callback`. |
| State | Per-session state with `output_key` for persisted results; state delta merging. |
| Deployment | Vertex AI Agent Engine, Cloud Run, GKE; `adk run`, `adk web`, `adk eval`, `adk deploy`. |
| Evaluation | Trajectory-based eval with metrics and partner integrations. |

## What Joch wraps and what it does not

```text
                ┌───────────────── your code ─────────────────┐
                │   from google.adk.agents import LlmAgent    │
                │   from google.adk.runners import Runner     │
                │   triage = LlmAgent(name=..., tools=[...])  │
                │   for ev in Runner(triage).run_async(...):  │
                │       ...                                   │
                └──┬───────────────┬───────────────┬──────────┘
                   │ FunctionTool  │ MCPToolset    │ callbacks
                   │ AgentTool     │ OpenAPIToolset│ (before_tool, ...)
                   ▼               ▼               ▼
   ── Joch boundary ─────────────────────────────────────────────
                   ▼
         ┌──────────────────────────────────────────────┐
         │ Joch google-adk FrameworkAdapter             │
         │   wraps tools (Function/Agent/OpenAPI)       │
         │   replaces MCPToolset with gateway client    │
         │   routes Gemini / Vertex calls via Router    │
         │   captures Events + state deltas to Trace    │
         │   bridges Session to Conversation            │
         │   maps before_tool_callback to toolCallReq   │
         │   keeps after_tool_callback for SDK logic    │
         └──────────────────────────────────────────────┘
```

Joch does **not** replace `LlmAgent`, the workflow agents, or the `Runner`. Joch adds a governed boundary around tools, MCP, models, memory, and observability — and contributes to AgBOM.

## Mapping table

| Google ADK | Joch |
|---|---|
| `LlmAgent(...)` | One [`Agent`](../specs/kubernetes/agent.md) record with `framework.adapterRef: google-adk` |
| `SequentialAgent`, `ParallelAgent`, `LoopAgent` | Recorded as `Agent.framework.workflow=sequential|parallel|loop`; each child `LlmAgent` has its own record |
| `FunctionTool(...)` | One [`Tool`](../specs/kubernetes/tool.md) record per tool; routed through the [Tool Gateway](../architecture/tool-gateway.md) |
| `AgentTool(agent)` | Modeled as a [`Handoff`](../specs/kubernetes/handoff.md) at runtime |
| `LongRunningFunctionTool` | Same as `Tool` plus `sideEffects.longRunning: true` |
| `OpenAPIToolset` | Generates one `Tool` per operation; gateway proxies the REST call |
| `MCPToolset` | Replaced with gateway-routed configuration; references [`MCPServer`](../specs/kubernetes/mcpserver.md) records |
| `GoogleSearchTool`, `BuiltInCodeExecutor` | Recognized by name; gateway applies side-effect classes (`read_only` / `code_execution`) |
| `SessionService` | Bridged to [`Conversation`](../specs/kubernetes/conversation.md) + canonical event log |
| `BaseMemoryService` | Bridged to Joch [`Memory`](../specs/kubernetes/memory.md) where the operator chooses to delegate |
| `BaseArtifactService` | Bridged to Joch [`Artifact`](../specs/kubernetes/artifact.md) |
| `Event` | Mirrored into Joch trace events |
| Callbacks (`before_*`, `after_*`) | `before_*` map to AOS request hooks; `after_*` continue to run in-process |
| `output_key` | Recorded in the trace and the `Conversation` final state |
| `adk eval` | Composable with Joch [`Eval`](../specs/kubernetes/eval.md); operator chooses native eval, Joch eval, or both |
| `adk deploy` | Joch's [`Deployment`](../specs/kubernetes/deployment.md) reconciles to the same target (Agent Engine, Cloud Run, GKE) via the runtime adapter |

## The smallest install

### Before — your existing code

```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService

def search_zendesk(query: str) -> list[dict]:
    ...

triage = LlmAgent(
    name="support_triage",
    model="gemini-2.5-pro",
    instruction="Triage tickets and draft replies.",
    tools=[FunctionTool(search_zendesk)],
)

runner = Runner(agent=triage, session_service=InMemorySessionService(), app_name="support")
async for event in runner.run_async(user_id="alice", session_id="sess-1", new_message="Refund for order 12345"):
    print(event)
```

### After — Joch-governed

```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from joch.adapters.google_adk import bind_runtime  # adapter package

def search_zendesk(query: str) -> list[dict]:
    ...

triage = LlmAgent(
    name="support_triage",
    model="gemini-2.5-pro",
    instruction="Triage tickets and draft replies.",
    tools=[FunctionTool(search_zendesk)],
)

runner = bind_runtime(
    Runner(agent=triage, session_service=InMemorySessionService(), app_name="support"),
    joch_agent_ref="support-triage",
    server_url="http://joch-server:8080",
)

async for event in runner.run_async(user_id="alice", session_id="sess-1", new_message="Refund for order 12345"):
    print(event)
```

`bind_runtime(...)` injects:

- `before_tool_callback` and `after_tool_callback` that emit AOS `steps/toolCallRequest` and `steps/toolCallResult`,
- `before_model_callback` and `after_model_callback` that emit `steps/message` and route the call through the Joch model router,
- `before_agent_callback` that emits `steps/agentTrigger` (and resolves the `ModelRoute`),
- a `Plugin` that mirrors `Event`s into Joch trace and bridges `state` deltas into `Conversation` events,
- replacement for any `MCPToolset` so MCP traffic flows through the [Joch MCP Gateway](../architecture/mcp-gateway.md).

## Workflow agents

`SequentialAgent` / `ParallelAgent` / `LoopAgent` are expressed in Joch as:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-pipeline
spec:
  framework:
    adapterRef: { name: google-adk }
    workflow: sequential
  children:
    - { name: collect-evidence }
    - { name: synthesize }
    - { name: review }
```

Each child has its own `Agent` record. The trace stitches them via `parent_tool_use_id`-like links in the `joch.parent.execution.id` attribute.

## Tools through the gateway

`FunctionTool`, `LongRunningFunctionTool`, `AgentTool`, `OpenAPIToolset`, and `MCPToolset` are all recognized:

- function tools execute in-process after the gateway approves,
- OpenAPI tools call out via the gateway with side-effect classification per operation,
- agent tools become `Handoff` records,
- MCP tools route through the MCP gateway for pinning, scanning, and audit.

## Sessions, memory, artifacts

The native `SessionService`, `BaseMemoryService`, and `BaseArtifactService` are kept. Joch additionally writes:

- a portable [`Conversation`](../specs/kubernetes/conversation.md) event log,
- [`Memory`](../specs/kubernetes/memory.md) records when the operator chooses to delegate memory to Joch's memory service,
- [`Artifact`](../specs/kubernetes/artifact.md) entries for outputs the agent emits.

`output_key` values are surfaced in the trace and stored as the canonical "final" content of the conversation.

## Models and Vertex routing

`model="gemini-2.5-pro"` and Vertex AI routing are recognized by the [Model Router](../architecture/model-router.md). The `ModelRoute` configured in the agent record adds capability checks, fallback, region / residency constraints, and budget enforcement.

## Deployment

`adk deploy` targets Vertex AI Agent Engine, Cloud Run, or GKE. Joch's [`Deployment`](../specs/kubernetes/deployment.md) reconciles to the same targets through the runtime adapter, so the deployment topology becomes:

```text
joch apply -f deployments/research-pipeline-prod.yaml
        │
        ▼
DeploymentController (kubernetes runtime adapter | cloud-run adapter | agent-engine adapter)
        │
        ▼
GKE | Cloud Run | Vertex AI Agent Engine
```

## Evaluation

`adk eval` produces trajectory-style results. Joch's [`Eval`](../specs/kubernetes/eval.md) can:

- consume the native ADK eval results as one metric, or
- run its own evals alongside, or
- gate `joch promote` on either or both.

## Migration matrix

| Existing | Joch step |
|---|---|
| `LlmAgent(...)` definitions | `joch discover --framework google-adk --path ./agents` produces stub records |
| `SessionService` choice | Keep it; the adapter writes a parallel Conversation log |
| `MCPToolset` | Replace with gateway-routed config; create `MCPServer` records |
| `OpenAPIToolset` | Auto-imported as `Tool` records; review side-effect classification |
| `adk deploy` to Cloud Run | Move to `joch apply -f deployments/...`; the cloud-run adapter handles deploy |
| Native `adk eval` | Keep; optionally compose with `joch eval` for promotion gates |

## What Joch leaves to the SDK

- The agent loop and decision-making.
- Workflow execution semantics (sequential / parallel / loop).
- The native eval framework's metric implementations.
- The `Event` shape returned to the caller.
- ADK-native plugin system for in-process extensions.

## Reference

- ADK home: <https://adk.dev/>
- Gemini Enterprise Agent Platform: <https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/adk>
- Tools and integrations: <https://adk.dev/tools-and-integrations>
- CLI: `adk run`, `adk web`, `adk eval`, `adk deploy`
