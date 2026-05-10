# Onboard from a Vendor SDK

If you already have agents in OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, or CrewAI, this is the path that brings them under Joch governance without rewriting them.

## The 30-minute path

1. **Install Joch** (`brew install joch && joch up`).
2. **Apply foundation records** — `FrameworkAdapter`, `Model`, `ModelRoute`, `Secret`.
3. **Discover existing agents** with `joch discover --framework <sdk> --path ...`.
4. **Review and apply** the stub `Agent` records.
5. **Wire the adapter into your code** with a single `bind_runtime(...)` / `joch_options(...)` / `.UseJoch(...)` call.
6. **Apply policies** — start with one (e.g., approval for `external_write`).
7. **Run a smoke test** and inspect the trace and AgBOM.

The detailed walkthroughs per SDK live in [Integrations](../integrations/index.md). What follows is the shared playbook.

## Decide what stays in your SDK

Joch deliberately leaves the agent loop, planner, prompts, structured outputs, and SDK-specific features to the SDK. Operators do **not** need to refactor:

- `Agent` definitions, instructions, tool function bodies (OpenAI Agents SDK).
- `query()` calls, `ClaudeAgentOptions` shape, hooks, subagents (Claude Agent SDK).
- `LlmAgent`, workflow agents, callbacks, `Runner`, native eval (Google ADK).
- `AIAgent`, `AgentThread`, middleware, workflows, OTel (Microsoft Agent Framework).
- `StateGraph`, nodes, edges, checkpointer (LangGraph).
- `Agent` / `Task` / `Crew` / `Process` (CrewAI).

## Decide what comes to Joch

These five things move into Joch records, regardless of SDK:

| | What it is in your SDK | What it becomes in Joch |
|---|---|---|
| 1. The agent identity | An `Agent` / `LlmAgent` / `AIAgent` / etc. instance | An [`Agent`](../specs/kubernetes/agent.md) record with `framework.adapterRef` |
| 2. Each tool the agent can call | A `function_tool` / `FunctionTool` / `AIFunction` / LangChain tool | One [`Tool`](../specs/kubernetes/tool.md) record per tool |
| 3. Each MCP server | An entry in `mcp_servers={...}` / `MCPToolset` / hosted MCP | One [`MCPServer`](../specs/kubernetes/mcpserver.md) record |
| 4. The model selection | A `model="..."` argument | A [`ModelRoute`](../specs/kubernetes/model-route.md) referenced by the agent record |
| 5. The guardrails / approvals you already have | SDK-specific guardrails, hooks, permission modes | One or more [`Policy`](../specs/kubernetes/policy.md) records, applied at the gateway boundary |

## Run discovery

```bash
joch discover --framework openai-agents-sdk    --path ./services
joch discover --framework claude-agent-sdk     --path ./coding-agents
joch discover --framework google-adk           --path ./pipelines
joch discover --framework ms-agent-framework   --path ./apps
joch discover --framework langgraph            --path ./graphs
joch discover --framework crewai               --path ./crews
```

Each command produces stub `Agent` records, plus draft `Tool` and `MCPServer` records derived from the SDK's metadata. Owners review, fill in `ModelRoute` and `Policy`, and `joch apply -f .`.

## Wire one line in your code

Each SDK has a single, minimal call that turns the SDK's outbound traffic into Joch-governed traffic. The exact form lives on the SDK page:

- **OpenAI Agents SDK** → [`bind_runtime(agent, joch_agent_ref=..., server_url=...)`](../integrations/openai-agents-sdk.md#the-smallest-install)
- **Claude Agent SDK** → [`joch_options(options, joch_agent_ref=..., server_url=...)`](../integrations/claude-agent-sdk.md#the-smallest-install)
- **Google ADK** → [`bind_runtime(runner, joch_agent_ref=..., server_url=...)`](../integrations/google-adk.md#the-smallest-install)
- **Microsoft Agent Framework** → [`.UseJoch(new JochOptions { ... })` or `bind_runtime(agent, ...)`](../integrations/microsoft-agent-framework.md#the-smallest-install-net)
- **LangGraph** → [`bind_runtime(app, joch_agent_ref=..., server_url=...)`](../integrations/langgraph.md#the-smallest-install)
- **CrewAI** → [`bind_runtime(crew, joch_agent_ref=..., server_url=...)`](../integrations/crewai.md#the-smallest-install)

That is the only code change in most cases.

## Verify

```bash
joch get agents
joch run <your-agent> "<smoke test>"
joch trace last
joch agbom <your-agent>
joch get toolcalls --agent <your-agent> --since 5m
joch denials ls --since 5m
```

If a tool call routed through the gateway, a model call routed through the router, the trace shows AOS hook decisions, and the AgBOM lists the right components — the onboarding is done.

## Move policies in over time

It is fine to start with **one** policy and expand. Common adoption order:

1. Approval required for any `external_write` or `financial` tool.
2. Cost cap per run and per day.
3. PII redaction in tool arguments.
4. Trust-score floor for MCP servers.
5. Eval-gate on `joch promote ... --to prod`.

Each is one resource. Each composes with the SDK's existing in-process guardrails — Joch does not replace them.

## What if my SDK is not listed?

Use the [custom-python](../integrations/custom-code.md#python) or [custom-typescript](../integrations/custom-code.md#typescript) adapter. The contract is one entrypoint function plus the `JochRuntime` client. You keep your loop; Joch keeps the boundary.
