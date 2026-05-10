# SDK Integrations

Joch is **not** an agent SDK. The pages in this section show, for each supported framework, exactly how Joch attaches to the SDK's native primitives — what the SDK keeps, what Joch governs, what changes in your code, and what does not.

## What Joch keeps in the SDK

Every supported SDK already does these jobs well, and Joch leaves them alone:

```text
Agent definition and prompts
Agent loop / planner
Tool registration
Provider client (model calls)
Native streaming and structured outputs
Native tracing surface inside the SDK process
SDK-specific session / thread shape
```

## What Joch adds at the boundary

Joch does **not** replace agent code. It wraps the **outbound boundary** that every SDK eventually crosses:

```text
Tool calls            → Joch Tool Gateway        AOS hooks: toolCallRequest / toolCallResult
MCP traffic           → Joch MCP Gateway         AOS hooks: protocols/MCP outbound + inbound
Model calls           → Joch Model Router        AOS hooks: agentTrigger / message
Memory operations     → Joch Memory service      AOS hooks: memoryContextRetrieval / memoryStore
RAG retrievals        → Joch RAG service         AOS hooks: knowledgeRetrieval
A2A messages          → Joch A2A Broker          AOS A2A hook surface
Trace events          → Joch Trace               OTLP + OCSF export
AgBOM                 → Joch AgBOM service       CycloneDX / SPDX / SWID
Approvals + Policy    → Joch Policy Engine       portable Policy resource
```

The wrapping happens through a per-SDK [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md), which is published as a separate package and tested against a conformance suite.

## Supported SDKs

<div class="grid cards" markdown>

-   :material-robot-outline: **OpenAI Agents SDK**

    ---

    `Agent`, `Runner`, function tools, `MCPServer`, handoffs, guardrails, sessions, tracing, `RunContextWrapper`, lifecycle hooks.

    [Read the integration](openai-agents-sdk.md)

-   :material-creation: **Claude Agent SDK**

    ---

    `query()`, `ClaudeAgentOptions`, built-in tools (`Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`, `WebSearch`, `WebFetch`, `Monitor`, `AskUserQuestion`), MCP, subagents, hooks, permission modes, sessions, skills, slash commands, plugins.

    [Read the integration](claude-agent-sdk.md)

-   :material-google: **Google ADK**

    ---

    `LlmAgent`, `SequentialAgent`, `ParallelAgent`, `LoopAgent`, `FunctionTool`, `AgentTool`, `MCPToolset`, `OpenAPIToolset`, `Session`/`SessionService`, `BaseMemoryService`, `Runner`, `Events`, callbacks, deployment to Vertex AI Agent Engine / Cloud Run / GKE.

    [Read the integration](google-adk.md)

-   :material-microsoft-azure: **Microsoft Agent Framework**

    ---

    `AIAgent`, `ChatAgent`, `AgentThread`, `AIFunction`, hosted MCP tools, A2A, OpenAPI integration, durable workflows, middleware, OpenTelemetry telemetry, Azure AI Foundry deployment.

    [Read the integration](microsoft-agent-framework.md)

-   :material-graph-outline: **LangGraph**

    ---

    `StateGraph`, `Node`, `Edge`, `Checkpointer`, `interrupt_before` / `interrupt_after`, tools as nodes, MCP via tool node, streaming.

    [Read the integration](langgraph.md)

-   :material-account-multiple: **CrewAI**

    ---

    `Agent`, `Task`, `Crew`, `Process`, tool integration, hierarchical / sequential processes.

    [Read the integration](crewai.md)

-   :material-code-tags: **Custom Code**

    ---

    A minimal entrypoint contract for home-grown Python or TypeScript agents that do not use a published SDK.

    [Read the integration](custom-code.md)

</div>

## What "integration" means concretely

Each integration page contains:

1. **The SDK in 60 seconds** — the primitives we care about, by name, with one-line semantics.
2. **What Joch wraps and what it does not** — boundary diagram in operator language.
3. **Mapping table** — every SDK primitive ↔ Joch resource, hook, or gateway.
4. **The smallest possible install** — actual code: SDK imports + Joch wiring, before / after.
5. **Tools** — how function / hosted / MCP tools route through the Joch tool gateway.
6. **MCP** — how MCP servers are registered, pinned, scanned, and proxied.
7. **Models** — how model calls route through the Joch model router and `ModelRoute`.
8. **Memory and RAG** — how the SDK's memory or retrieval surface attaches to Joch.
9. **State** — how SDK sessions / threads / checkpoints flow into [`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md).
10. **Hooks and tracing** — which AOS hooks fire, and how the SDK's native tracing composes with Joch.
11. **Approvals** — how a `deny` / `modify` decision surfaces in the SDK's runtime.
12. **AgBOM** — what shows up in the per-agent AgBOM for an SDK-based agent.
13. **Migration matrix** — onboarding steps for an existing SDK app.
14. **What we leave to the SDK** — features Joch deliberately does not touch.
15. **Reference** — links to the SDK's own docs and API reference.
