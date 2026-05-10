# Microsoft Agent Framework Integration

The [**Microsoft Agent Framework**](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview) is the direct successor to Semantic Kernel and AutoGen — agents plus graph-based workflows, with first-class .NET and Python SDKs and Azure AI Foundry integration. Joch attaches to it through the `ms-agent-framework` framework adapter.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `AIAgent` (.NET) / `agent` (Python) | The agent abstraction returned by `client.AsAIAgent(...)` / `client.as_agent(...)`. |
| `AgentThread` | Session-based conversation state. |
| `AIFunction` | Function-tool definition. |
| Hosted MCP tools | First-class MCP support via the framework's MCP client / hosted MCP integration. |
| A2A | Agent-to-agent protocol support (send / stream / cancel / get task / notification config). |
| OpenAPI tools | OpenAPI spec → tool. |
| Workflows | Graph-based, type-safe routing with checkpointing and human-in-the-loop. `WorkflowExecutor`, edges, type-routed flow. |
| Middleware | Intercept agent / model / tool actions; analogous to ASP.NET-style middleware. |
| Telemetry | OpenTelemetry traces, logs, metrics out of the box. |
| Providers | Microsoft Foundry, Anthropic, Azure OpenAI, OpenAI, Ollama, and more. |
| Packages | `.NET`: `Microsoft.Agents.AI.Foundry` (and friends). `Python`: `agent-framework`. |
| Memory | Context providers for agent memory. |
| Deployment | Azure AI Foundry hosted, or any container target. |

## What Joch wraps and what it does not

```text
                ┌─────────────── your code ────────────────┐
                │  using Microsoft.Agents.AI;              │
                │  AIAgent agent = client.AsAIAgent(...);  │
                │  await agent.RunAsync("...");            │
                └──┬───────────────┬───────────────┬───────┘
                   │ AIFunction    │ MCP / A2A     │ middleware
                   ▼               ▼               ▼
   ── Joch boundary ─────────────────────────────────────────────
                   ▼
         ┌──────────────────────────────────────────────┐
         │ Joch ms-agent-framework FrameworkAdapter     │
         │   middleware: emit AOS hooks at each stage   │
         │   wrap MCP / A2A clients with Joch gateways  │
         │   route Foundry/OpenAI/Azure/Anthropic       │
         │     calls through Joch Model Router          │
         │   bridge AgentThread to Conversation         │
         │   forward OTel spans to Joch Trace           │
         └──────────────────────────────────────────────┘
```

Joch does **not** replace `AIAgent`, `AgentThread`, workflows, OpenAPI integration, the durable workflow execution model, or middleware. Joch sits inside the middleware pipeline and at the outbound boundary.

## Mapping table

| Microsoft Agent Framework | Joch |
|---|---|
| `AIAgent` (`.NET`) / `agent` (`Python`) | One [`Agent`](../specs/kubernetes/agent.md) record with `framework.adapterRef: ms-agent-framework` |
| `AgentThread` | Bridged to [`Conversation`](../specs/kubernetes/conversation.md) + canonical event log |
| `AIFunction` | One [`Tool`](../specs/kubernetes/tool.md) record per function; routed through the [Tool Gateway](../architecture/tool-gateway.md) |
| Hosted MCP tools | One [`MCPServer`](../specs/kubernetes/mcpserver.md) record per server; traffic via [MCP Gateway](../architecture/mcp-gateway.md) |
| A2A protocol | [`Handoff`](../specs/kubernetes/handoff.md) records; A2A hook surface enforced |
| OpenAPI integration | One `Tool` record per OpenAPI operation; gateway proxies REST |
| Workflows | Recorded as `Agent.framework.workflow.shape=graph` with edges and checkpoints; `Execution` state captures progress per node |
| Middleware | Joch ships an `AgentMiddleware` package that emits AOS hooks |
| Telemetry (OpenTelemetry) | Mirrored into Joch [`Trace`](../specs/kubernetes/trace.md); native OTel spans are augmented with `joch.*` attributes |
| Providers (Foundry, Azure OpenAI, OpenAI, Anthropic, Ollama) | All routed through the [Model Router](../architecture/model-router.md) |
| Memory context providers | Bridged to [`Memory`](../specs/kubernetes/memory.md) when delegated |
| Azure AI Foundry deployment | A first-class Joch `Deployment` adapter targets Foundry |

## The smallest install (.NET)

### Before

```csharp
using Azure.AI.Projects;
using Azure.Identity;
using Microsoft.Agents.AI;

AIAgent agent = new AIProjectClient(
        new Uri("https://your-foundry-service.services.ai.azure.com/api/projects/your-foundry-project"),
        new AzureCliCredential())
    .AsAIAgent(
        model: "gpt-5.4-mini",
        instructions: "You are a friendly assistant.");

Console.WriteLine(await agent.RunAsync("What is the largest city in France?"));
```

### After — Joch-governed

```csharp
using Azure.AI.Projects;
using Azure.Identity;
using Microsoft.Agents.AI;
using Joch.Adapters.MicrosoftAgentFramework;  // Joch.Adapters.MicrosoftAgentFramework NuGet

AIAgent agent = new AIProjectClient(
        new Uri("https://your-foundry-service.services.ai.azure.com/api/projects/your-foundry-project"),
        new AzureCliCredential())
    .AsAIAgent(
        model: "gpt-5.4-mini",
        instructions: "You are a friendly assistant.")
    .UseJoch(new JochOptions
    {
        AgentRef = "support-triage",
        ServerUrl = "http://joch-server:8080",
    });

Console.WriteLine(await agent.RunAsync("What is the largest city in France?"));
```

`UseJoch(...)` registers middleware that:

- emits AOS `steps/agentTrigger` on run start,
- emits `steps/toolCallRequest` and `steps/toolCallResult` around each `AIFunction` invocation,
- routes the inner model client through the Joch model router (preserving Foundry / OpenAI / Anthropic provider choice),
- mirrors OTel spans into Joch trace with `joch.*` attributes,
- bridges `AgentThread` events into [`Conversation`](../specs/kubernetes/conversation.md).

## The smallest install (Python)

### Before

```python
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential

credential = AzureCliCredential()
client = FoundryChatClient(
    project_endpoint="https://your-foundry-service.services.ai.azure.com/api/projects/your-foundry-project",
    model="gpt-5.4-mini",
    credential=credential,
)

agent = client.as_agent(
    name="HelloAgent",
    instructions="You are a friendly assistant.",
)

result = await agent.run("What is the largest city in France?")
print(f"Agent: {result}")
```

### After — Joch-governed

```python
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from joch.adapters.ms_agent_framework import bind_runtime

credential = AzureCliCredential()
client = FoundryChatClient(
    project_endpoint="https://your-foundry-service.services.ai.azure.com/api/projects/your-foundry-project",
    model="gpt-5.4-mini",
    credential=credential,
)

agent = client.as_agent(
    name="HelloAgent",
    instructions="You are a friendly assistant.",
)

agent = bind_runtime(
    agent,
    joch_agent_ref="support-triage",
    server_url="http://joch-server:8080",
)

result = await agent.run("What is the largest city in France?")
print(f"Agent: {result}")
```

## Workflows

Microsoft Agent Framework workflows are graph-based with type-safe routing, checkpointing, and human-in-the-loop. Joch records workflow runs as:

- one root `Execution` per workflow run,
- one child `Execution` per agent node,
- `Handoff` records on type-routed edges,
- `Approval` records at human-in-the-loop checkpoints,
- `StateCheckpoint` entries at workflow checkpoints.

Durable execution semantics in the native framework are preserved; Joch only observes and governs.

## Middleware

The framework's middleware is the natural extension point for Joch. Joch ships an `AgentMiddleware` (`.NET`) and a Python equivalent that:

- registers AOS hook emission,
- enforces `Policy` decisions,
- wraps inner clients (model / MCP / A2A / OpenAPI),
- composes safely with your own middleware (your middleware runs alongside Joch's; Joch never silently replaces it).

## A2A

The framework's A2A surface is bound to the [Joch A2A broker](../architecture/data-plane.md), which records `A2AMessageSent` / `A2AMessageReceived` events and applies the AOS A2A hook surface (`send_message`, `stream_message`, `cancel_request`, `get_task`, notification config get/set/resubscribe).

## OpenTelemetry

Microsoft Agent Framework emits OTel spans natively. The Joch adapter:

- adds `joch.*` attributes (`joch.agent.name`, `joch.framework`, `joch.policy.id`, `joch.tenant.id`),
- adds Joch-specific events (`joch.policy.denied`, `joch.budget.exceeded`, `joch.provider.switched`),
- forwards spans to the Joch trace endpoint in addition to your existing OTel collector — both work in parallel.

See the [OpenTelemetry Mapping](../aos/opentelemetry-mapping.md).

## AgBOM

The agent's [`AgBOM`](../specs/kubernetes/agbom.md) lists:

- `joch.framework=ms-agent-framework`, the .NET / Python package versions,
- every `AIFunction` and OpenAPI operation,
- every hosted MCP server,
- the active provider (Foundry / Azure OpenAI / OpenAI / Anthropic / Ollama),
- middleware configured (with `joch.middleware.<name>`),
- workflow shape if applicable.

## Migration matrix

| Existing | Joch step |
|---|---|
| `AsAIAgent(...)` / `client.as_agent(...)` | Add `.UseJoch(...)` (`.NET`) or `bind_runtime(...)` (Python) |
| `AIFunction`s | Auto-discovered as `Tool` records during `joch discover` |
| Hosted MCP tools | Register `MCPServer` records; the adapter routes traffic through the gateway |
| `AgentThread` | Continues to work; the adapter writes a parallel Conversation log |
| Workflow with checkpointing | Joch records workflow checkpoints as `StateCheckpoint`; durable replay is preserved by the native runtime |
| OpenTelemetry already in place | Continue your existing exporter; Joch trace is additive |
| Foundry deployment | Use Joch's `Deployment` adapter targeting Foundry, or continue native deployment and observe through Joch |

## What Joch leaves to the SDK

- The agent loop and middleware execution model.
- The graph-workflow execution and checkpointing.
- Type-safe routing semantics inside workflows.
- Native middleware semantics for non-AOS concerns (validation, retries, formatting).
- Microsoft Foundry-specific deployment and management features.

## Reference

- Overview: <https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview>
- Blog announcement: <https://azure.microsoft.com/en-us/blog/introducing-microsoft-agent-framework/>
- Migration from Semantic Kernel: <https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/>
- Migration from AutoGen: <https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/>
- Transparency FAQ: <https://github.com/microsoft/agent-framework/blob/main/TRANSPARENCY_FAQS.md>
