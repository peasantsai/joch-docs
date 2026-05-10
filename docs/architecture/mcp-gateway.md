# MCP Gateway

The MCP gateway is the registry, version-pinning, and firewall layer for every Model Context Protocol server an agent talks to. Joch does not "support MCP" the way an SDK supports MCP. Joch makes MCP **safe and governable** at fleet scale.

## Why a gateway is needed

Every modern SDK is adopting MCP. Anyone can publish an MCP server. The result: agents transitively connect to a long tail of community servers with no shared trust model. Reported MCP risks include local-process execution, prompt-injection in tool results, supply-chain poisoning, and unsafe stdio handling. ([MCP](https://modelcontextprotocol.io/specification/2025-11-25))

A control plane needs answers to:

- which MCP servers are in use, by which agents, in which environments,
- what tools each server exposes, with what schemas, at what version,
- which servers have been schema-drift checked since the last apply,
- which servers are pinned, deprecated, or quarantined,
- which credentials each server requires, and how they are scoped,
- which MCP messages a Guardian Agent has denied or modified,
- how MCP results have been scanned and redacted before reaching the agent.

The MCP gateway gives those answers and enforces the corresponding policies.

## Position in the architecture

```text
Agent (any SDK using MCP)
        │  outbound MCP message
        ▼
[ Joch MCP Gateway ]
        │  - resolves MCPServer record (must exist in registry)
        │  - validates against pinned tool schemas
        │  - enforces Policy via Policy Engine (allow / deny / modify)
        │  - hooks: protocols/MCP outbound (toolCallRequest if it is a tool call)
        │  - emits AOS trace events
        │  - calls the MCP server (or local server in sandbox)
        │  - hook: protocols/MCP inbound (toolCallResult)
        │  - applies result redaction / prompt-injection scanning
        ▼
Trusted MCP server (registered, pinned, scanned)
```

The agent never reaches an unregistered MCP server. The MCP traffic flows through the gateway in both directions.

## Registry and discovery

Each MCP server is an [`MCPServer`](../specs/kubernetes/mcpserver.md) record:

```yaml
apiVersion: joch.dev/v1alpha1
kind: MCPServer
metadata:
  name: github
spec:
  transport: streamable_http
  endpoint:
    url: https://mcp.example.com/github
  auth:
    type: oauth2
    secretRef: { name: github-mcp-oauth }
  exposes:
    tools: true
    resources: true
    prompts: false
  discovery:
    enabled: true
    refreshInterval: 10m
  security:
    sandbox: true
    allowStdio: false
    pinServerVersion: true
    pinnedVersion: 1.2.0
    trustedPublisher: github
  policies:
    - name: mcp-tool-safety-policy
status:
  phase: Ready
  discoveredTools:
    - github.search_issues
    - github.create_issue
  discoveredResources:
    - github://repos
  lastDiscoveryAt: "2026-05-10T10:00:00Z"
  trustScore: 0.92
  schemaDrift: false
```

Discovery refresh checks for new tools, removed tools, and **schema drift**. Drift is surfaced as an event and may auto-quarantine the server depending on policy.

## Version pinning

By default, every MCP server is pinned to a specific version. Upgrades are explicit:

```bash
joch mcp ls
joch describe mcpserver github
joch mcp upgrade github --to 1.3.0 --review
joch mcp pin github --to 1.2.0
```

Pinning protects against silent supply-chain changes between deploys.

## Trust scoring

A `trustScore` is computed from:

- whether the server is published by an allow-listed publisher,
- whether it ships a signed manifest,
- whether the schema has been stable across N discoveries,
- the AOS trace history (denial rate, error rate, latency),
- third-party allow-list integrations (e.g., your security team's MCP allow-list).

Policies can require a minimum trust score for production use.

## Sandboxing

For MCP servers that run as local processes (stdio or local HTTP):

- the gateway runs them under cgroups / seccomp profiles,
- network egress is constrained to the gateway's allow-list,
- filesystem access is restricted to a per-server scratch directory,
- environment variables come from the secret broker, never from the agent.

This converts the "any binary can be an MCP server" property from a risk into a governed runtime path.

## AOS hooks (Instrument)

The MCP gateway implements the OWASP AOS hook contract for `protocols/MCP`:

| Direction | Hook | When |
|---|---|---|
| Outbound | `protocols/MCP` (outbound) | before sending a JSON-RPC message to the server |
| Inbound | `protocols/MCP` (inbound) | before forwarding a server response to the agent |

Each call returns `allow`, `deny`, or `modify` per AOS. See [Hooks](../aos/hooks.md).

If the outbound message is a `tools/call`, the gateway also emits `steps/toolCallRequest` for parity with the [Tool Gateway](tool-gateway.md). Inbound results emit `steps/toolCallResult`. This makes MCP tool calls indistinguishable from native tool calls in policy and audit.

## Result scanning

Inbound MCP messages are scanned for prompt-injection patterns. Patterns can be specified in policy or sourced from third-party feeds. A scan hit can:

- modify the result to strip the injection,
- deny the result entirely,
- annotate the result with a warning the agent can take into account,
- create an incident.

## AgBOM contribution

Every MCP server contributes to the per-agent [AgBOM](../specs/kubernetes/agbom.md). The AgBOM lists each server, its pinned version, its discovered capabilities, its publisher, and its current trust score. Operators inspect the AgBOM to answer "what does this agent depend on, and is any of it risky?"

## Operator commands

```bash
joch mcp ls
joch mcp add github --endpoint https://mcp.example.com/github
joch mcp discover github
joch mcp approve-tool github.create_issue
joch mcp quarantine suspicious-server --reason "schema drift"
joch mcp policy set github --read-only
joch mcp upgrade github --to 1.3.0
joch mcp trust github
```

## Why this is the strongest wedge

Every SDK in the market is rapidly adding MCP support. As that adoption widens, the lack of cross-SDK MCP governance becomes more visible. Joch's MCP gateway is the place where:

- security teams can require MCP allow-lists,
- platform teams can pin versions and detect drift,
- compliance teams can prove what each agent depends on,
- operators can quarantine a bad server in seconds, fleet-wide.

That combination — registry + firewall + AOS hooks + AgBOM + sandboxing + drift detection — does not exist in any single SDK and is hard to retrofit. It is therefore both a sharp **first wedge** and a durable **defensible position**.
