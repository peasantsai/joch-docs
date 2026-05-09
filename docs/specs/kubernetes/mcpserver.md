# MCPServer

Kubernetes-style YAML resource specification for Joch `MCPServer` resources.

[Back to Kubernetes specs](index.md)

## MCPServer spec

MCP servers expose tools, resources, and prompts to clients. Official MCP docs describe servers as offering features like resources, prompts, and tools. ([Model Context Protocol][1])

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
    secretRef:
      name: github-mcp-oauth

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
    commandWhitelist: []
    pinServerVersion: true
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
  lastDiscoveryAt: "2026-05-09T10:00:00Z"
```

I would be strict with MCP security. Recent reporting has highlighted MCP risks around local process execution, prompt injection, supply-chain poisoning, and unsafe STDIO handling, so the spec should make transport, trust, sandboxing, and command restrictions explicit. ([Tom's Hardware][4])

---

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
