# Govern an MCP Server

This walkthrough takes about 10 minutes. You will register an MCP server, pin a version, scan inbound results for prompt injection, and quarantine the server on demand.

## 1. Register the server

Save as `mcp-github.yaml`:

```yaml
apiVersion: tools.joch.dev/v1alpha1
kind: MCPServer
metadata: { name: github }
spec:
  transport: streamable_http
  endpoint:
    url: https://mcp.example.com/github
  auth:
    type: oauth2
    secretRef: { name: github-mcp-oauth }
  exposes: { tools: true, resources: true, prompts: false }
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
  scanning:
    promptInjectionScan: true
  policies:
    - name: mcp-tool-safety
```

```bash
joch apply -f mcp-github.yaml
joch mcp ls
joch describe mcpserver github
```

The discovery output lists the server's tools, resources, and version, plus the computed trust score.

## 2. Add the policy

Save as `mcp-policy.yaml`:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata: { name: mcp-tool-safety }
spec:
  appliesTo:
    agents: { selector: { matchLabels: { env: prod } } }
  rules:
    - when: { mcpServer.trustScore: "<0.75" }
      action: { deny: true, reason: trust-score-too-low }
    - when: { mcpServer.schemaDrift: true }
      action: { deny: true, reason: schema-drift-detected }
    - when: { mcpResult.injectionScan.hit: true }
      action: { modify: redact, reason: prompt-injection-stripped }
```

```bash
joch apply -f mcp-policy.yaml
```

## 3. Use it from an agent

Reference the server from an agent record (any framework):

```yaml
spec:
  framework:
    adapterRef: { name: openai-agents-sdk }
    pythonModule: my_agents.coder:agent
  mcpServers:
    - name: github
```

```bash
joch apply -f agent.yaml
joch run coder "Open the latest issue in acme/joch and summarize."
```

The trace captures every outbound JSON-RPC and inbound result through `protocols/MCP` hooks. Inbound results are scanned for prompt-injection patterns before reaching the agent.

## 4. Pin and upgrade safely

```bash
joch mcp upgrade github --to 1.3.0 --review
```

The review surfaces the schema diff (added tools, removed tools, changed schemas). Apply with:

```bash
joch mcp pin github --to 1.3.0
```

Until you pin, the server stays at `1.2.0` for every agent that uses it.

## 5. Quarantine on incident

Schema drift, denial spike, or trust-score drop:

```bash
joch mcp quarantine github --reason "schema drift at 03:14 UTC"
```

The quarantine takes effect fleet-wide immediately. Every agent that depends on `github` receives a denial reason for the duration of the quarantine.

## 6. Audit

```bash
joch get toolcalls --mcpserver github --since 24h
joch trace exec-20260510-001
joch denials ls --policy mcp-tool-safety --since 24h
joch agbom ls --using-mcpserver github
joch agbom support-triage --diff --from 16 --to 17
```

The AgBOM diff makes MCP version changes auditable.

## Next

- [Onboard from a vendor SDK](migration-from-vendor-sdk.md)
- [MCP Gateway architecture](../architecture/mcp-gateway.md)
- [`MCPServer` reference](../specs/kubernetes/mcpserver.md)
