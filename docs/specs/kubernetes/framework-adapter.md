# FrameworkAdapter

A `FrameworkAdapter` connects a Joch [`Agent`](agent.md) record to the SDK that actually runs the agent. There is one `FrameworkAdapter` resource per supported framework — agents reference it by name.

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: FrameworkAdapter
metadata:
  name: openai-agents-sdk
  labels:
    framework: openai-agents-sdk
spec:
  framework: openai-agents-sdk
  description: >
    Adapter for OpenAI's Agents SDK (Python and TypeScript flavors).
    Wires the SDK's tool surface to the Joch tool gateway, MCP traffic
    to the Joch MCP gateway, and model calls to the Joch model router.

  language: python
  adapterImage: ghcr.io/peasantsai/joch-adapter-openai:1.0.0
  sdkMinVersion: "0.6.0"

  capabilities:
    toolBinding: true
    mcpBinding: true
    modelRouterBinding: true
    streaming: true
    a2a: false
    structuredOutput: true

  bindings:
    toolGatewayUrl: http://joch-tool-gateway
    mcpGatewayUrl: http://joch-mcp-gateway
    modelRouterUrl: http://joch-router

  conformance:
    suite: ghcr.io/peasantsai/joch-adapter-conformance:1.0
    lastPassedAt: "2026-05-08T14:00:00Z"

status:
  phase: Ready
  agentsBound: 7
```

## Adapter contract

Every adapter implements the same conceptual interface:

```ts
interface FrameworkAdapter {
  resolve(record: AgentRecord, manifest: CompiledAgentManifest): Promise<RunnableAgent>;
  run(execution: ExecutionRecord): AsyncIterable<TraceEvent>;
  bindTools(gatewayUrl: string, tools: ToolBinding[]): void;
  bindMcp(gatewayUrl: string, servers: McpBinding[]): void;
  bindModelRouter(routerUrl: string, route: ModelRouteRef): void;
  capabilities(): CapabilitySet;
}
```

The exact language form varies (Python protocol class, TypeScript interface, Go interface). The semantics are identical across implementations.

## Initial supported frameworks

```text
openai-agents-sdk        OpenAI Agents SDK
claude-agent-sdk         Claude Agent SDK
google-adk               Google Agent Development Kit
ms-agent-framework       Microsoft Agent Framework
langgraph                LangGraph (LangChain ecosystem)
crewai                   CrewAI
custom-python            arbitrary Python module exposing joch_entrypoint(...)
custom-typescript        arbitrary TypeScript module exposing jochEntrypoint(...)
```

A `FrameworkAdapter` resource is published per framework. Adding a new framework is a package, not a fork of Joch core.

## Conformance

Every adapter must pass the conformance suite before being marked `Ready`:

```text
agent registers correctly
tool calls route through tool gateway
MCP traffic routes through MCP gateway
model calls route through model router with ModelRoute fallback
trace events emit with correct attributes
policy denials surface to the SDK user as the SDK's native error type
approvals pause and resume execution as expected
```

The `conformance` block tracks the suite reference and the last successful run.

## Capability vector

The `capabilities` block declares what the adapter supports. The compiler validates this against the agent's `spec` at apply time:

```text
toolBinding              can bind tool calls through the gateway
mcpBinding               can route MCP through the gateway
modelRouterBinding       can route model calls through the router
streaming                end-to-end streaming responses
a2a                      Agent-to-Agent protocol
structuredOutput         JSON / schema-constrained outputs
vision                   image inputs
audio                    audio inputs
```

If an `Agent` requires a capability the adapter does not declare, the apply rejects with a clear diagnostic.

[Back to the catalog](index.md)
