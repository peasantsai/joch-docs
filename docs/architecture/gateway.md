# Gateway

Joch Gateway is the MVP wedge. It sits between agents and the tools or MCP servers they call.

## Flow

```text
Agent
  -> Joch Gateway
  -> Policy check
  -> Approval if required
  -> MCP server / API / tool
  -> Audit log
  -> Agent
```

## Responsibilities

```text
proxy MCP and generic HTTP tools
normalize tool-call requests
classify side-effect level
evaluate policy
pause calls that require approval
redact sensitive arguments
filter risky results
write audit records
return the tool result to the agent
```

## Side-effect levels

```text
read_only
local_write
external_write
code_execution
financial
privileged
unknown
```

The side-effect level drives the default policy posture. `unknown` should be treated as risky until reviewed.

## MCP-specific controls

```text
scan server capabilities
record exposed tools
capture input/output schemas
diff schemas over time
classify new capabilities
quarantine changed or untrusted servers
block unsafe transports by policy
```

## Audit output

Each call creates or updates:

```text
ToolCall
Approval, when required
TraceEvent entries
Agent risk metadata
ABOM inputs
```

Gateway adoption is the moment Joch AI becomes more than documentation. It is the point where governance is enforced.
