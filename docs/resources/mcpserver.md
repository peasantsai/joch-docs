# MCPServer

`MCPServer` records a Model Context Protocol server registered with Joch Gateway.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: MCPServer
metadata:
  name: slack
spec:
  transport: streamable_http
  endpoint: https://mcp.example.com/slack
  owner:
    team: platform
  trustStatus: reviewed
  scanning:
    enabled: true
    onSchemaDrift: quarantine
status:
  phase: Ready
  toolsExposed:
    - slack.send_message
    - slack.read_channel
  lastScan: "2026-05-11T00:00:00Z"
  riskLevel: high
```

## Required scan outputs

```text
server name
transport
endpoint
tools exposed
schema versions
trust status
last scan
owner
risk level
```

## Drift behavior

Schema drift should produce a `TraceEvent` and an operator-visible diff. Policy can decide whether changed servers are allowed, denied, or quarantined.
