# Tool

A `Tool` resource describes a single callable function exposed through the [tool gateway](../../architecture/tool-gateway.md). Tools may be functions in an agent's code, REST endpoints, MCP tools (registered via [`MCPServer`](mcpserver.md)), or sandboxed OS commands.

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Tool
metadata:
  name: zendesk.create_ticket
  namespace: support-platform
spec:
  type: rest

  description: Create a Zendesk support ticket on behalf of a customer.

  endpoint:
    method: POST
    url: https://example.zendesk.com/api/v2/tickets.json
    auth:
      type: bearer
      secretRef:
        name: zendesk-api-token

  inputSchema:
    type: object
    required: [subject, body]
    properties:
      subject:
        type: string
      body:
        type: string
      priority:
        type: string
        enum: [low, normal, high, urgent]

  outputSchema:
    type: object
    properties:
      ticketUrl: { type: string }
      ticketId:  { type: integer }

  sideEffects:
    level: external_write
    idempotent: false
    requiresApproval: true
    idempotencyKeyTemplate: "zendesk-{{ args.subject | hash }}"

  safety:
    allowedAgents:
      - support-triage
      - support-escalation
    deniedNamespaces:
      - sandbox
    rateLimit:
      callsPerMinute: 10
      callsPerDay: 500

  observability:
    logArgs: true
    logResult: true
    redactFields:
      - args.body
      - args.customer_email

status:
  phase: Ready
  totalCalls7d: 412
  failureRate7d: 0.014
```

## Tool types

```text
function       a callable in user code, registered via a framework adapter
rest           a REST endpoint with input/output schemas
mcp            a tool exposed by an MCPServer (registered with the MCP gateway)
os             a sandboxed command executed by Joch (e.g., file or shell ops)
```

For `type: mcp`, set `mcpServerRef` instead of `endpoint`:

```yaml
spec:
  type: mcp
  mcpServerRef:
    name: github
  description: Create a GitHub issue.
  inputSchema: {...}
```

## Side-effect class

`sideEffects.level` drives default behavior:

```text
read_only           reads external state; no writes; no costs
local_write         writes to local agent-owned state (working memory, sandbox file)
external_write      writes to external systems (issue trackers, ticketing, Slack)
financial           moves money or commits to charges
communication       sends messages to humans (email, SMS, Slack DM)
code_execution      executes code (sandboxed REPL, browser automation)
privileged          touches sensitive infrastructure (cluster admin, DB DDL)
```

Each level has default approval policies and audit defaults. Operators override per record.

## Idempotency

`sideEffects.idempotencyKeyTemplate` lets Joch derive a stable key from arguments. The tool gateway deduplicates calls with the same key within a configurable window, and prevents duplicate side effects across [provider migrations](../../architecture/state-portability.md).

## Safety

```text
allowedAgents       only listed agents may call the tool
deniedNamespaces    no agent in these namespaces may call the tool
rateLimit           hard caps per minute / day, enforced at the gateway
```

Tool safety is composable with [`Policy`](policy.md). Tool-level rules are minimum guarantees; policies layer additional rules on top.

[Back to the catalog](index.md)
