# Resources

Joch AI uses Kubernetes-style YAML records for the MVP resource model.

```yaml
apiVersion: joch.ai/v1alpha1
kind: <Kind>
metadata:
  name: string
  namespace: string
  labels: {}
spec: {}
status: {}
```

## MVP resources

<div class="grid cards" markdown>

-   :material-account-cog-outline: **Agent**

    ---

    Framework-agnostic record of an agent and its owner, runtime, models, tools, MCP servers, policies, and risk level.

    [Open spec](agent.md)

-   :material-tools: **Tool**

    ---

    Callable capability with schemas, source, side-effect level, owner, and approval requirement.

    [Open spec](tool.md)

-   :material-server-security: **MCPServer**

    ---

    Registered MCP server with transport, endpoint, tools exposed, trust status, scan state, and risk.

    [Open spec](mcpserver.md)

-   :material-shield-key-outline: **Policy**

    ---

    Portable rules that allow, deny, modify, or require approval for tool/MCP action.

    [Open spec](policy.md)

-   :material-text-box-search-outline: **ToolCall**

    ---

    One tool invocation with input, side-effect class, approval requirement, and result.

    [Open spec](toolcall.md)

-   :material-account-check-outline: **Approval**

    ---

    Human decision record for risky action.

    [Open spec](approval.md)

-   :material-package-variant-closed: **ABOM**

    ---

    Agent Bill of Materials for models, tools, MCP servers, policies, secrets referenced, risk flags, and deployments.

    [Open spec](abom.md)

-   :material-cogs: **Execution**

    ---

    One concrete run of an agent.

    [Open spec](execution.md)

-   :material-timeline-text-outline: **TraceEvent**

    ---

    Durable event record for discovery, policy, tool-call, approval, execution, and ABOM activity.

    [Open spec](trace-event.md)

</div>
