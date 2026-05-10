# Anthropic Provider

The Anthropic provider routes calls to the Messages API. Used by every Joch agent that selects an `anthropic:*` model, plus by the [Claude Agent SDK](../integrations/claude-agent-sdk.md) integration when run with native Anthropic API auth.

## Authentication

Native API:

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: anthropic-api-key }
spec:
  source: { type: env, name: ANTHROPIC_API_KEY }
```

Third-party routing:

```text
CLAUDE_CODE_USE_BEDROCK=1   # AWS Bedrock
CLAUDE_CODE_USE_VERTEX=1    # Google Vertex AI
CLAUDE_CODE_USE_FOUNDRY=1   # Microsoft Foundry
```

The router records the resolved provider in trace events.

## Capability vector

```yaml
capabilities:
  text: true
  vision: true
  audio: false
  toolCalling: true
  structuredOutput: true
  jsonSchema: false       # use input_schema on tools, not strict response_format
  streaming: true
  reasoning: true         # extended thinking on supported models
  computerUse: true       # computer-use beta on supported models
  embeddings: false
```

## Tool calling

Anthropic's `tools=[...]` and tool-use blocks map to Joch's [`ToolCall`](../specs/kubernetes/toolcall.md). The adapter:

- forwards tool definitions with their `input_schema`,
- intercepts `tool_use` blocks in `assistant` messages,
- executes through the [Tool Gateway](../architecture/tool-gateway.md) and replies with `tool_result` blocks.

## Prompt caching

`cache_control` is preserved end-to-end. `usage.cache_creation_input_tokens` and `usage.cache_read_input_tokens` are surfaced on Joch trace events as separate counters.

## Extended thinking

`thinking={"type": "enabled", "budget_tokens": ...}` is forwarded. `ThinkingBlock` deltas are streamed through the trace as annotations on the `ModelCall` span; they are not exposed as user-visible content unless the agent opts in.

## Streaming

Server-Sent Events. The adapter forwards chunks; cumulative usage is recorded on the parent span at completion.

## Region / residency

Anthropic native API does not publish regional residency. For residency-bound deployments, use Bedrock (region constrained) or Vertex AI (region constrained), or use a local Claude-compatible model.

## Fallback behavior

The router falls forward off Anthropic on 5xx, 429, capability mismatch, budget breach, or extended-thinking error (`thinking.type.enabled`-type errors require Agent SDK v0.2.111+ and provider feature parity).

## Cost reporting

Computed from `Model.spec.pricing`. The Anthropic adapter additionally tracks cache reads vs. writes for accurate cost attribution under prompt caching.

## Known limits

- Tool result content sometimes arrives with attacker-controlled text from external systems; the [MCP Gateway](../architecture/mcp-gateway.md) and the [Tool Gateway](../architecture/tool-gateway.md) scan for prompt-injection patterns before forwarding.
- `JSONSchema strict` mode is not supported the same way as OpenAI; the adapter declares `jsonSchema: false` so capability checks select OpenAI / Foundry when strict JSON Schema is required.

## Reference

- API: <https://docs.claude.com/en/api>
- Agent SDK overview: <https://code.claude.com/docs/en/agent-sdk/overview>
