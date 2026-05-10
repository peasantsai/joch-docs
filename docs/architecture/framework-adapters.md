# Framework Adapters

Framework adapters are the bridge between an agent record in Joch and the SDK that actually runs the agent. There is one adapter per supported framework. Adapters live in their own packages so that adding support for a new framework does not touch the Joch core.

## Why adapters exist

Each agent SDK has its own runtime expectations: how agents are started, how tools are registered, how messages are passed, how state is persisted, how MCP is wired in, how observability is exported. An adapter normalizes those expectations onto Joch's gateways, hooks, and resource model.

The result: one Joch [`Agent`](../specs/kubernetes/agent.md) record can run against any supported framework, and policies / inventory / observability work uniformly across frameworks.

## Supported frameworks (initial set)

```text
openai-agents-sdk        OpenAI Agents SDK (Python and JS)
claude-agent-sdk         Claude Agent SDK (Python and TypeScript)
google-adk               Google Agent Development Kit
ms-agent-framework       Microsoft Agent Framework
langgraph                LangGraph (LangChain ecosystem)
crewai                   CrewAI
custom-python            arbitrary Python module exposing an entrypoint
custom-typescript        arbitrary TypeScript module exposing an entrypoint
```

New adapters land as additional packages without breaking older agent records.

## Adapter contract

An adapter implements a single, language-agnostic interface:

```ts
interface FrameworkAdapter {
  // Resolve a Joch Agent record to a runnable agent in the framework.
  resolve(record: AgentRecord, manifest: CompiledAgentManifest): Promise<RunnableAgent>;

  // Run one execution: takes input, emits events to Joch trace, returns output.
  run(execution: ExecutionRecord): AsyncIterable<TraceEvent>;

  // Wire the framework's tool surface to the Joch tool gateway.
  bindTools(gatewayUrl: string, tools: ToolBinding[]): void;

  // Wire MCP traffic through the Joch MCP gateway.
  bindMcp(gatewayUrl: string, servers: McpBinding[]): void;

  // Route model calls through the Joch model router.
  bindModelRouter(routerUrl: string, route: ModelRouteRef): void;

  // Capability vector for compatibility checks (used by ModelRoute and migration).
  capabilities(): CapabilitySet;
}
```

The exact language form of this interface differs per ecosystem (Python protocol class, TypeScript interface, Go interface). The semantics are identical.

## How an agent connects

An agent record references its adapter through a [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md) resource:

```yaml
apiVersion: joch.dev/v1alpha1
kind: FrameworkAdapter
metadata:
  name: openai-agents-sdk
spec:
  framework: openai-agents-sdk
  adapterImage: ghcr.io/peasantsai/joch-adapter-openai:1.0.0
  language: python
  capabilities:
    toolBinding: true
    mcpBinding: true
    modelRouterBinding: true
    streaming: true
    a2a: false
```

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: support-triage
spec:
  framework:
    adapterRef:
      name: openai-agents-sdk
    entrypoint: ./agents/support_triage.py
    pythonModule: support.agents.triage:agent
  …
```

At apply time, the [compiler](control-plane.md#compiled-agent-manifests) resolves the adapter, validates capabilities (e.g., the agent depends on streaming, the adapter supports it), and emits a CompiledAgentManifest the runtime worker can execute.

## What an adapter does at runtime

```text
1. The runtime worker pulls an Execution off the queue.
2. The worker fetches the CompiledAgentManifest.
3. The adapter loads the user's agent code via the entrypoint.
4. The adapter wires:
     tool surface  →  joch-tool-gateway
     MCP surface   →  joch-mcp-gateway
     model surface →  joch-model-router
     memory ops    →  joch-memory
     RAG ops       →  joch-rag
6. The adapter runs the agent loop (the SDK's loop, not Joch's).
7. The adapter translates SDK events into Joch trace events.
8. On completion or failure, the adapter writes the Execution status.
```

The agent code itself is unchanged. Adapter wiring is invisible to the SDK user.

## Discovery

`joch discover` is adapter-aware. For each supported framework:

```bash
joch discover --framework openai-agents-sdk --path ./services
joch discover --framework claude-agent-sdk --path ./coding-agents
joch discover --framework langgraph --path ./pipelines
```

The adapter scans the path, extracts agent definitions, and proposes Joch records. The owner reviews the diff, fills in policies, model routes, and budgets, and applies.

## Custom code adapters

For organizations with home-grown agent code, the `custom-python` and `custom-typescript` adapters require only an entrypoint that exposes:

```python
# Python
def joch_entrypoint(execution_context: dict, joch: JochRuntime) -> dict:
    ...
```

```typescript
// TypeScript
export async function jochEntrypoint(
  ctx: ExecutionContext,
  joch: JochRuntime,
): Promise<ExecutionResult> {
  // ...
}
```

The adapter handles the rest: tool binding, MCP binding, model routing, observability, and policy enforcement.

## Adapter testing

Each adapter has an interoperability test suite that runs against the live Joch gateways with a sample agent:

```text
adapter-conformance:
  agent registers correctly
  tool calls route through tool gateway
  MCP traffic routes through MCP gateway
  model calls route through model router with ModelRoute fallback
  trace events emit with correct attributes
  policy denials surface to the SDK user as the SDK's native error type
  approvals pause and resume execution as expected
```

A new adapter is considered supported only after it passes the conformance suite.

## What adapters do *not* do

- Adapters do not re-implement SDK features.
- Adapters do not store conversation state. They translate SDK events into [`Conversation`](../specs/kubernetes/conversation.md) writes.
- Adapters do not bypass gateways. All cross-boundary traffic goes through Joch's data plane.

This keeps adapters small (typically a few hundred lines per framework) and focused.
