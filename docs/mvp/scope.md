# MVP Scope

## Included

```text
Joch CLI
Joch Server
Joch Gateway
Joch Agent Registry
Joch Tool Registry
Joch MCP Registry
Policy Engine v0
Tool-call audit log
Approval workflow v0
ABOM generator v0
Trace/event log v0
Docker Compose deployment
Kubernetes Helm deployment
OpenAI Agents SDK adapter
Claude Agent SDK adapter
LangGraph adapter
CrewAI adapter
Generic HTTP/MCP proxy mode
```

## Not included

```text
custom agent framework
custom graph runtime
custom multi-agent planner
full hosted SaaS
advanced eval system
advanced RAG platform
full model migration
advanced memory lifecycle
marketplace
complex billing
```

## Core surfaces

| Surface | Purpose |
|---|---|
| `joch` CLI | Developer/operator interface for init, apply, discover, gateway, policy, approvals, ABOM, trace, and get/describe. |
| `joch-server` | API server and registry for agents, tools, MCP servers, policies, approvals, audit records, and events. |
| `joch-gateway` | MCP/tool proxy, policy authorization, approval pause/resume, redaction, result filtering, and audit logging. |
| `joch-worker` | Discovery, MCP scanning, ABOM generation, background indexing, and event processing. |

## Success shape

An existing agent calls a tool through Joch Gateway. Joch classifies the tool, evaluates policy, requires approval for risky actions, executes only after approval, and records the full audit trail.
