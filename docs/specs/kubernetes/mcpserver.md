# MCPServer

An `MCPServer` registers a Model Context Protocol server with the [MCP gateway](../../architecture/mcp-gateway.md). The MCP gateway proxies, pins, scans, and audits every interaction with the server. Agents never reach an unregistered MCP server.

[Back to the catalog](index.md)

## Spec

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
    onSchemaDrift: quarantine

  security:
    sandbox: true
    allowStdio: false
    pinServerVersion: true
    pinnedVersion: 1.2.0
    trustedPublisher: github
    minTrustScore: 0.75

  policies:
    - name: mcp-tool-safety-policy

  scanning:
    promptInjectionScan: true
    redactFieldsInResults:
      - sshKeys
      - apiTokens

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
  callCount7d: 1822
  denyCount7d: 4
```

## Why MCP needs a gateway

Recent reports have highlighted MCP risks: local-process execution, prompt-injection in tool results, supply-chain poisoning, and unsafe stdio. Joch's gateway addresses each:

- `pinServerVersion` defends against silent version drift.
- `sandbox: true` constrains local-process MCP servers via cgroups / seccomp.
- `allowStdio: false` denies unsafe stdio transports unless explicitly enabled.
- `scanning.promptInjectionScan` inspects inbound results before forwarding to the agent.
- `policies` is the same portable [`Policy`](policy.md) surface used elsewhere — uniform across SDKs.

## Discovery

`discovery.refreshInterval` controls how often the gateway re-fetches the server's capabilities. A change in the discovered tool list (or in tool schemas) is **schema drift**; the policy in `discovery.onSchemaDrift` decides whether to alert, deny, or auto-quarantine.

## Trust score

`trustScore` is composed from publisher allow-listing, signed manifests, schema stability, denial / error history, and any third-party allow-list. Policies can require a minimum trust score.

## ABOM contribution

Every `MCPServer` contributes to the per-agent [`ABOM`](abom.md). The ABOM lists each server, its pinned version, its discovered capabilities, its publisher, and its trust score at the time of generation.

## Operator commands

```bash
joch mcp ls
joch describe mcpserver github
joch mcp upgrade github --to 1.3.0 --review
joch mcp pin github --to 1.2.0
joch mcp quarantine suspicious-server --reason "schema drift"
```

[Back to the catalog](index.md)
