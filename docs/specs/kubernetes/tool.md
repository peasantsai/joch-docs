# Tool

Kubernetes-style YAML resource specification for Joch `Tool` resources.

[Back to Kubernetes specs](index.md)

## Tool spec

Tools are callable functions. MCP tools are the obvious compatibility target: MCP tools have names and input schemas and let models interact with external systems. ([Model Context Protocol][3])

```yaml
apiVersion: joch.dev/v1alpha1
kind: Tool
metadata:
  name: github.create_issue
spec:
  type: mcp
  mcpServerRef:
    name: github

  description: Create a GitHub issue.

  inputSchema:
    type: object
    required:
      - repo
      - title
      - body
    properties:
      repo:
        type: string
      title:
        type: string
      body:
        type: string

  outputSchema:
    type: object
    properties:
      issueUrl:
        type: string
      issueNumber:
        type: integer

  sideEffects:
    level: external_write
    idempotent: false
    requiresApproval: true

  safety:
    allowedAgents:
      - engineering-agent
      - triage-agent
    deniedNamespaces:
      - sandbox
    rateLimit:
      callsPerMinute: 10

  observability:
    logArgs: true
    logResult: true
    redactFields:
      - token
      - password

status:
  phase: Ready
```

I would explicitly classify tools:

```text
read_only
local_write
external_write
financial
communication
code_execution
privileged
```

This matters for governance and approvals.

---

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
