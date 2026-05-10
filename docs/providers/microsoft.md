# Microsoft Foundry / Azure OpenAI Provider

The Microsoft provider supports Azure AI Foundry projects, Azure OpenAI deployments, and Microsoft Agent Framework runtimes.

## Authentication

Entra-ID via `AzureCliCredential`, `DefaultAzureCredential`, managed identity, or service principal:

```bash
az login
export AZURE_FOUNDRY_PROJECT_ENDPOINT=https://your-foundry-service.services.ai.azure.com/api/projects/your-foundry-project
```

For service principals:

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: azure-foundry-sp }
spec:
  source: { type: azure-key-vault, name: ... }
```

## Capability vector

```yaml
capabilities:
  text: true
  vision: true
  audio: true
  toolCalling: true
  structuredOutput: true
  jsonSchema: true
  streaming: true
  reasoning: true
  computerUse: true
  embeddings: true
```

## Tool calling

Foundry preserves OpenAI semantics for tool calling. The adapter forwards `tools` and `tool_choice`, applies AOS `toolCallRequest` and `toolCallResult` hooks, and routes hosted MCP through the [MCP Gateway](../architecture/mcp-gateway.md).

## Region / residency

Foundry projects are bound to a region at creation. The adapter records the region on every `ModelCall` span and refuses to dispatch from a non-matching `dataResidency` `Environment`.

## Fallback behavior

Falls forward on 5xx, 429, throttling, capability mismatch, residency mismatch, and budget breach.

## Cost reporting

Foundry returns token counts; cost is computed from `Model.spec.pricing`. Customer-billable usage flows through Foundry's usage records; Joch's reporting is additive and reconciles per-agent / per-team / per-execution.

## Microsoft Agent Framework integration

When the agent's framework adapter is `ms-agent-framework`, the adapter additionally:

- bridges middleware to Joch hooks,
- mirrors OpenTelemetry spans to Joch trace,
- composes the framework's workflow checkpoints with Joch [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md).

See [Microsoft Agent Framework Integration](../integrations/microsoft-agent-framework.md).

## Known limits

- Some Foundry-only features (e.g., Azure-specific connectors) require resource-level allow-listing on the Azure side; the adapter surfaces failures with provider error codes preserved.

## Reference

- Foundry overview: <https://learn.microsoft.com/en-us/azure/ai-foundry/>
- Microsoft Agent Framework: <https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview>
