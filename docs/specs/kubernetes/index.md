# Resource Catalog

One page per Joch resource kind. All pages use the same Kubernetes-style envelope (`apiVersion / kind / metadata / spec / status`).

<div class="grid cards" markdown>

-   :material-account-cog-outline: **Agent**

    ---

    The framework-agnostic record of an agent: identity, model policy, tools, MCP servers, memory bindings, and policies.

    [Open spec](agent.md)

-   :material-puzzle-outline: **FrameworkAdapter**

    ---

    Connects an agent record to the SDK that runs it (OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, custom).

    [Open spec](framework-adapter.md)

-   :material-brain: **Model**

    ---

    A model backend record: provider, capabilities, limits, pricing, defaults.

    [Open spec](model.md)

-   :material-router-network-wireless: **ModelRoute**

    ---

    Capability-aware, cost-aware, region-aware fallback policy across providers.

    [Open spec](model-route.md)

-   :material-tools: **Tool**

    ---

    A callable function exposed through the tool gateway, with side-effect classification and safety controls.

    [Open spec](tool.md)

-   :material-server-security: **MCPServer**

    ---

    A Model Context Protocol server registered with the MCP gateway: discovery, version pinning, sandboxing, trust scoring.

    [Open spec](mcpserver.md)

-   :material-tools: **ToolCall**

    ---

    One concrete invocation of a tool, with idempotency key, side-effect class, approval status, and result.

    [Open spec](toolcall.md)

-   :material-shield-key-outline: **Policy**

    ---

    Portable policy-as-code for tool, model, network, data, budget, and audit rules.

    [Open spec](policy.md)

-   :material-account-check-outline: **Approval**

    ---

    A human-review record for risky tool calls or release promotions.

    [Open spec](approval.md)

-   :material-format-list-bulleted-square: **AgBOM**

    ---

    Agent Bill of Materials extending OWASP AgBOM (CycloneDX, SPDX, SWID).

    [Open spec](agbom.md)

-   :material-graph-outline: **Trace**

    ---

    Per-execution event log, exporting OpenTelemetry and OCSF.

    [Open spec](trace.md)

-   :material-cogs: **Execution**

    ---

    One concrete run of an agent.

    [Open spec](execution.md)

-   :material-message-text-outline: **Conversation**

    ---

    Vendor-neutral, durable record of an agent's dialog.

    [Open spec](conversation.md)

-   :material-content-save-check-outline: **StateCheckpoint**

    ---

    Vendor-neutral mid-conversation snapshot used for provider migration.

    [Open spec](state-checkpoint.md)

-   :material-database-outline: **Memory**

    ---

    Working / semantic / episodic memory bindings.

    [Open spec](memory.md)

-   :material-database-search-outline: **RAG**

    ---

    Retrieval-augmented generation indices.

    [Open spec](rag.md)

-   :material-book-outline: **KnowledgeSource**

    ---

    Pointers to corpora that feed RAG indices.

    [Open spec](knowledge-source.md)

-   :material-paperclip: **Artifact**

    ---

    Durable execution outputs.

    [Open spec](artifact.md)

-   :material-key-variant: **Secret**

    ---

    External secret references; values resolved at the gateway boundary.

    [Open spec](secret.md)

-   :material-cash-multiple: **Budget**

    ---

    Cost and usage caps enforced before model and tool calls.

    [Open spec](budget.md)

-   :material-clipboard-check-outline: **Eval**

    ---

    Scored agent evaluation with thresholds and release gates.

    [Open spec](eval.md)

-   :material-rocket-launch-outline: **Deployment**

    ---

    Where and how many instances of an agent run.

    [Open spec](deployment.md)

-   :material-domain: **Environment**

    ---

    Promotion boundary (`dev`, `staging`, `prod`) with bound policies and budgets.

    [Open spec](environment.md)

-   :material-account-multiple-outline: **Team / Namespace**

    ---

    Multi-tenant ownership boundary.

    [Open spec](team-namespace.md)

-   :material-arrow-decision-outline: **Handoff**

    ---

    Inter-agent transfer of control (A2A).

    [Open spec](handoff.md)

</div>
