# MCP Governance

> A security team needs to know every MCP server the agent fleet depends on, pin versions, scan results for prompt injection, and quarantine anything suspicious — without touching the SDKs each team uses.

## Symptoms

- A new MCP server adopted by one team is silently picked up by three others.
- An MCP server publishes a schema change overnight; agents start failing in unexpected ways.
- A tool result returns attacker-controlled text that the agent treats as instructions.
- An incident requires answering "which agents talked to this server in the last 24 hours" within minutes.

## What Joch does

The [MCP Gateway](../architecture/mcp-gateway.md) registers, pins, scans, and proxies every MCP server. Each interaction emits AOS hooks (`protocols/MCP` outbound and inbound) and trace events.

## Walkthrough

### 1. Register a server

```bash
joch mcp add github --endpoint https://mcp.example.com/github --auth oauth2
joch mcp pin github --to 1.2.0
```

The corresponding [`MCPServer`](../specs/kubernetes/mcpserver.md) record carries discovery, pinning, sandboxing, and policy settings.

### 2. Inspect capabilities

```bash
joch mcp ls
joch describe mcpserver github
joch mcp discover github
```

The discovery output lists tools, resources, and prompts the server exposes, along with the trust score and any schema drift.

### 3. Apply policies

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: mcp-tool-safety
spec:
  appliesTo:
    agents: { selector: { matchLabels: { env: prod } } }
  rules:
    - when:
        mcpServer.trustScore: "<0.75"
      action: { deny: true, reason: trust-score-too-low }
    - when:
        mcpServer.schemaDrift: true
      action: { deny: true, reason: schema-drift-detected }
    - when:
        mcpResult.injectionScan.hit: true
      action: { modify: redact, reason: prompt-injection-stripped }
```

```bash
joch apply -f policies/mcp-tool-safety.yaml
```

### 4. Quarantine and investigate

```bash
joch mcp quarantine suspicious-server --reason "schema drift detected at 03:14"
joch get toolcalls --mcpserver suspicious-server --since 24h
joch trace exec-20260510-001
joch denials ls --policy mcp-tool-safety --since 24h
```

A quarantined server is taken out of rotation immediately, fleet-wide. The trace history reconstructs every interaction.

### 5. Audit the dependency graph

```bash
joch abom ls --using-mcpserver github
joch abom support-triage --diff --from 16 --to 17
```

ABOM diffs make MCP changes auditable: which version was pinned, when, by whom, with what trust score.

## Resources involved

- [`MCPServer`](../specs/kubernetes/mcpserver.md)
- [`Tool`](../specs/kubernetes/tool.md), [`ToolCall`](../specs/kubernetes/toolcall.md)
- [`Policy`](../specs/kubernetes/policy.md)
- [`Trace`](../specs/kubernetes/trace.md), [`ABOM`](../specs/kubernetes/abom.md)

## Outcome

The security team operates a single MCP control plane: registry, version pinning, scanning, sandboxing, drift detection, and quarantine. Application teams continue to use their SDKs unchanged; the gateway is invisible to them until a policy denies or modifies a request.
