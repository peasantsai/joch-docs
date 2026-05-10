# OpenAI Provider

The OpenAI provider routes calls to the OpenAI API: `chat.completions` and the newer `responses` API. Used by every Joch agent that selects an `openai:*` model.

## Authentication

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: openai-api-key }
spec:
  source:
    type: env
    name: OPENAI_API_KEY
```

The router resolves the key at the call boundary; values never enter the resource store, the trace, or the AgBOM.

For Azure OpenAI deployments, use the [Microsoft Foundry / Azure OpenAI](microsoft.md) provider instead.

## Capability vector

```yaml
capabilities:
  text: true
  vision: true
  audio: true             # for gpt-realtime / multimodal models
  toolCalling: true
  structuredOutput: true
  jsonSchema: true
  streaming: true
  reasoning: true         # for `o*` and `gpt-5-thinking`
  computerUse: true       # for computer-use-preview models
  embeddings: true
```

## Tool calling

OpenAI's tool calls map 1:1 to Joch's [`ToolCall`](../specs/kubernetes/toolcall.md). The Joch adapter:

- forwards `tools=[...]` and `tool_choice=...` to the provider,
- applies AOS `steps/toolCallRequest` to the inputs before submission,
- intercepts tool-use messages, executes them through the [Tool Gateway](../architecture/tool-gateway.md), and emits `steps/toolCallResult`.

## Structured outputs

`response_format={"type": "json_schema", "json_schema": {...}}` is preserved end-to-end. Schema validation runs after the provider response and before the agent receives it.

## Streaming

Chunked Server-Sent Events. The Joch adapter forwards chunks unchanged, with `joch.modelroute.fallback_index` carried as a span attribute on the parent span (not per chunk).

## Region / residency

OpenAI does not publish regional endpoints with strong residency guarantees. To enforce EU residency, use Azure OpenAI through the [Microsoft Foundry](microsoft.md) provider, or use a local model.

## Fallback behavior

Joch falls forward off OpenAI when:

- the API returns 5xx, 429 (rate-limit), or 408,
- the model rejects the request as unsupported (capability mismatch),
- a `Budget` rule fires.

Failover behavior is governed by the [`ModelRoute`](../specs/kubernetes/model-route.md) `failover` block.

## Cost reporting

The provider returns token counts. Joch computes `costUsd` from the configured `Model.spec.pricing` block:

```yaml
pricing:
  currency: USD
  inputPerMillionTokens: 0
  outputPerMillionTokens: 0
```

Update the pricing block when OpenAI changes prices; the router uses the values active at the time of the call.

## Known limits

- Realtime / Voice models require the `realtime` adapter (separate from the standard chat adapter).
- The Responses API and Chat Completions API have slightly different semantics; the adapter normalizes both into the canonical state.

## Reference

- API: <https://platform.openai.com/docs/api-reference>
- Agents SDK guide: <https://developers.openai.com/api/docs/guides/agents>
