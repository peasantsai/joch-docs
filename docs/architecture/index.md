# Architecture

Joch is a control plane plus a thin set of data-plane gateways. The control plane owns desired state, policy, inventory, approvals, and release gates. The data plane owns the boundary between agents and the systems they talk to: tools, MCP servers, models, memory, RAG indices, and other agents.

## High-level shape

```text
                     ┌───────────────────────┐
                     │      joch CLI / UI    │
                     └──────────┬────────────┘
                                │
                                ▼
                     ┌───────────────────────┐
                     │  Joch Control Plane   │  desired state, policy, inventory
                     │  (apiserver, store,   │  release gates, approvals, AgBOM
                     │   policy, scheduler)  │
                     └──────────┬────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Tool Gateway    │  │  MCP Gateway     │  │  Model Router    │   data plane
│  AOS hooks       │  │  AOS hooks       │  │  AOS hooks       │
│  (toolCall*)     │  │  (protocols/MCP) │  │  (message)       │
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
         │                     │                     │
         ▼                     ▼                     ▼
   external APIs         MCP servers          provider APIs (OpenAI,
   functions             (registered,         Anthropic, Google,
   callable funcs        pinned, scanned)     local models)

         ▲                     ▲                     ▲
         │                     │                     │
┌────────┴─────────────────────┴─────────────────────┴────────┐
│           Framework Adapters (one per SDK)                   │
│   OpenAI Agents SDK | Claude Agent SDK | Google ADK |        │
│   Microsoft Agent Framework | LangGraph | CrewAI | custom    │
└──────────────────────────────────────────────────────────────┘
```

## Sections

<div class="grid cards" markdown>

-   :material-server-network: **Control Plane**

    ---

    The services that own desired state, policy, inventory, approvals, and release gates.

    [Read the control plane design](control-plane.md)

-   :material-pipe-valve: **Data Plane**

    ---

    The gateways and routers that sit on the boundary between agents and external systems.

    [Read the data plane design](data-plane.md)

-   :material-puzzle-outline: **Framework Adapters**

    ---

    How Joch attaches to OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, and custom code.

    [Read the framework adapter design](framework-adapters.md)

-   :material-tools: **Tool Gateway**

    ---

    The AOS-aligned enforcement point for every tool call, with side-effect classification, approvals, idempotency, and audit.

    [Read the tool gateway design](tool-gateway.md)

-   :material-server-security: **MCP Gateway**

    ---

    The registry, version-pinning, and firewall layer for every Model Context Protocol server an agent talks to.

    [Read the MCP gateway design](mcp-gateway.md)

-   :material-swap-horizontal: **State Portability**

    ---

    Vendor-neutral conversation state, migration checkpoints, and capability validation when switching providers.

    [Read the state portability design](state-portability.md)

-   :material-shield-key: **Policy Engine**

    ---

    Portable policy-as-code, the AOS Guardian Agent role, and where rules are enforced.

    [Read the policy engine design](policy-engine.md)

-   :material-router-network: **Model Router**

    ---

    Capability-aware, cost-aware, region-aware fallback across providers.

    [Read the model router design](model-router.md)

-   :material-shield-lock-outline: **Trust and Security Model**

    ---

    Trust zones, secret handling, network policies, prompt-injection mitigations, and the Joch security boundary.

    [Read the trust and security model](trust-and-security-model.md)

-   :material-server: **Service Architecture**

    ---

    Concrete services, deployment modes, APIs, packaging, and runtime adapters.

    [Read the service architecture](service-architecture.md)

</div>

## Architecture rules

These rules are non-negotiable across every Joch service:

1. **Agent code stays in the SDK.** Joch never re-implements an SDK's agent loop, planner, or memory primitives.
2. **Cross-SDK boundaries are governed.** Tool calls, MCP messages, model calls, memory reads/writes, and RAG retrievals always cross a Joch gateway.
3. **State is owned by Joch.** Vendor APIs are inference backends; conversation, memory, and artifact state live in Joch resources.
4. **Policy is portable.** A `Policy` resource is the same shape regardless of SDK and is enforced at the boundary.
5. **Hooks are AOS-compliant.** Every gateway exposes the OWASP AOS hook contract (`allow` / `deny` / `modify`).
6. **Records survive runtimes.** Local, Docker, Kubernetes, and managed runtimes all consume the same records via runtime adapters.
7. **Resources are diffable.** Every record is versioned and structurally diffable so that releases are auditable.
