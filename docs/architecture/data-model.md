# Data Model

The MVP data model keeps the resource set small and directly tied to the wedge.

## Core resources

```text
Agent
Tool
MCPServer
Policy
ToolCall
Approval
Execution
TraceEvent
ABOM
SecretRef
```

## Relationship map

```text
Agent
  -> Tools
  -> MCPServers
  -> Policies
  -> Executions
  -> ABOM

Execution
  -> ToolCalls
  -> Approvals
  -> TraceEvents

Tool
  -> MCPServer, when sourced from MCP
  -> ToolCalls
  -> Policies
```

## Record speeds

| Resource | Change pattern |
|---|---|
| Agent | Changes when discovered, registered, or edited by owner. |
| Tool | Changes when schema, source, owner, or side-effect classification changes. |
| MCPServer | Changes when endpoint, transport, trust, or discovered capabilities change. |
| Policy | Changes through operator review. |
| ToolCall | Created per invocation. |
| Approval | Created only when policy requires human decision. |
| Execution | Created per run. |
| TraceEvent | Appended during discovery, policy, gateway, approval, and execution events. |
| ABOM | Generated on demand and after dependency changes. |

## Event names

```text
agent.discovered
tool.discovered
mcp.scanned
policy.applied
toolcall.requested
toolcall.approved
toolcall.denied
toolcall.completed
execution.started
execution.completed
abom.generated
```

The event log is not a secondary feature. It is how Joch AI proves what happened.
