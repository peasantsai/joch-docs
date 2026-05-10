# Google Provider

The Google provider routes calls to Vertex AI / Gemini API. Used by Joch agents selecting `google:gemini-*` models, and by the [Google ADK](../integrations/google-adk.md) integration.

## Authentication

Service-account auth:

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: google-vertex-sa }
spec:
  source:
    type: gcp-secret-manager
    name: projects/example/secrets/vertex-sa-json
```

Or via env vars:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json
export GOOGLE_CLOUD_PROJECT=example
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
  computerUse: false
  embeddings: true
```

## Tool calling

Gemini function calling maps to Joch's [`ToolCall`](../specs/kubernetes/toolcall.md). The adapter forwards tool declarations and intercepts `functionCall` parts. ADK's `MCPToolset` is replaced by gateway-routed configuration so MCP traffic flows through the [MCP Gateway](../architecture/mcp-gateway.md).

## Region / residency

Vertex AI exposes regional endpoints (`us-central1`, `europe-west4`, `asia-southeast1`, …). Set the region on the [`Model`](../specs/kubernetes/model.md) record:

```yaml
routing:
  regions: [eu]
```

The router refuses to dispatch a Vertex call from an `EU` `dataResidency` `Environment` to a non-EU region.

## Fallback behavior

Falls forward on 5xx, 429, capability mismatch, region mismatch (when `dataResidency` is set), and budget breach.

## Cost reporting

Computed from `Model.spec.pricing`. Vertex bills per character for some models; the adapter normalizes to tokens for consistent reporting.

## Known limits

- Multimodal inputs require the `inlineData` / `fileData` shapes; the adapter handles the conversion from Joch's canonical content blocks.
- Tool calling and structured outputs have model-specific shape differences; the adapter selects the right shape per model id.

## Reference

- Vertex AI generative AI: <https://cloud.google.com/vertex-ai/generative-ai/docs>
- Gemini Enterprise Agent Platform: <https://docs.cloud.google.com/gemini-enterprise-agent-platform>
