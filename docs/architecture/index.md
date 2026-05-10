# Architecture

Joch AI is a control plane plus a tool/MCP data-plane gateway.

The control plane owns desired state, inventory, policy, approvals, audit records, and ABOMs. The gateway owns the boundary between agents and tools.

```text
                 +--------------+
                 |   joch CLI   |
                 +------+-------+
                        |
                        v
                 +--------------+
                 | joch-server  |
                 +------+-------+
                        |
        +---------------+----------------+
        v               v                v
+--------------+ +--------------+ +--------------+
| joch-worker  | | joch-gateway | |  datastore   |
+------+-------+ +------+-------+ +--------------+
       |                |
       v                v
 discovery        MCP servers/tools
 adapters         external APIs
```

## Architecture rules

1. Agent code stays in the original SDK.
2. Tool and MCP calls cross Joch Gateway.
3. Policy is evaluated before side effects.
4. Approval decisions are durable.
5. Audit records are written for requests, decisions, denials, approvals, and completions.
6. ABOM generation reads the same inventory records used by enforcement.
7. Local, Docker, and Kubernetes deployments expose the same API model.

## Pages

- [Services](services.md)
- [Gateway](gateway.md)
- [Policy and Approvals](policy-approvals.md)
- [Data Model](data-model.md)
- [Deployment](deployment.md)
- [Integrations](integrations.md)
