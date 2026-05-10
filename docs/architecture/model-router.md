# Model Router

The model router is Joch's provider-neutral, capability-aware, cost-aware, region-aware front door for every model call. It implements [`ModelRoute`](../specs/kubernetes/model-route.md) policies and exposes the OWASP AOS Instrument hooks for `message` and `agentTrigger`.

## Why a router

Each vendor wants you to call its provider directly. That is convenient until you need:

- portable cost accounting,
- cross-vendor fallback when a provider is degraded,
- capability-aware selection (tool calling, structured output, vision, long context, reasoning),
- region-aware routing for data residency,
- centralized prompt-injection scanning of inbound responses,
- policy-aware rejection of disallowed providers.

The router gives you those without changing agent code.

## Position in the architecture

```text
Agent (any SDK)
        │  message / completion request
        ▼
[ Joch Model Router ]
        │  - resolves ModelRoute
        │  - capability-checks the request against the candidate model
        │  - enforces Policy via Policy Engine (allow / deny / modify)
        │  - hooks: agentTrigger, message
        │  - calls the provider via its adapter
        │  - on failure, tries the next fallback
        │  - records ModelCallStarted / ModelCallCompleted
        │  - applies inbound scanning per Policy
        ▼
Provider API (OpenAI | Anthropic | Google | Microsoft Foundry | local | …)
```

## ModelRoute resolution

Agent records reference a `ModelRoute` rather than a single model:

```yaml
apiVersion: joch.dev/v1alpha1
kind: ModelRoute
metadata:
  name: research-default
spec:
  requirements:
    toolCalling: true
    contextWindowMin: 200000
    structuredOutput: true
  strategy:
    primary: openai:gpt-5-thinking
    fallback:
      - anthropic:claude-sonnet
      - google:gemini-pro
  constraints:
    maxCostUsdPerExecution: 5
    dataResidency: EU
```

At call time the router:

1. Selects the highest-priority candidate that satisfies `requirements`.
2. Enforces `constraints` against the running execution (cost so far, region of origin).
3. Asks the policy engine if the candidate is permitted in this context.
4. Calls the provider via its adapter.
5. On failure, tries the next candidate per `strategy`.

Failures recorded include explicit `5xx`, timeouts, capability validation failures, policy denials, and budget breaches.

## Capability matching

Capabilities are advertised by [`Model`](../specs/kubernetes/model.md) records:

```yaml
capabilities:
  text: true
  vision: true
  audio: false
  toolCalling: true
  structuredOutput: true
  jsonSchema: true
  streaming: true
  reasoning: true
  computerUse: false
  embeddings: false
```

The router validates that the **runtime** request — the tools attached, the response format expected, the modalities supplied — fits the candidate. If it does not, the router falls forward in `strategy` rather than silently degrading.

## Hooks

The router implements the AOS Instrument hooks for `agentTrigger` and `message` (user and agent). The Guardian Agent (the [Policy Engine](policy-engine.md) by default) can:

- block a trigger (e.g., a user prompt that violates policy),
- modify an outbound message (e.g., redact PII before it reaches the provider),
- block or modify an inbound response (e.g., scrub a prompt-injection attempt sourced from a tool result),
- halt the model call entirely.

See [Hooks](../aos/hooks.md).

## Cost accounting

Every `ModelCallCompleted` event includes:

```text
provider, model, inputTokens, outputTokens, costUsd, latencyMs, region
```

These attributes feed cost rollups (`joch cost by-team`, `joch top models`) and budget enforcement.

## Failover and idempotency

Failover is automatic per `ModelRoute.strategy`. The router enforces these guarantees:

- A request that has produced a side effect (a tool call already executed within the same turn) cannot fail over silently. The router consults the [tool gateway](tool-gateway.md) for in-flight side effects before retrying.
- For idempotent requests (no side effects yet), failover is safe and recorded as `ProviderSwitched`.
- A failover that exceeds `constraints` budgets denies; the agent receives a structured error.

## Streaming

The router supports streaming responses end-to-end. Hooks can inspect each chunk before forwarding to the agent. Modifications during streaming (e.g., on-the-fly redaction) are supported but constrained: the modified stream cannot lead the original; only delete and replace are permitted.

## Local models

Local models (llama.cpp, vLLM, Ollama, MLX, on-prem APIs) are first-class. They are described by `Model` records with an explicit `endpoint.type: local` and routed identically to hosted providers. This is essential for data-residency-bound deployments.

## Provider adapters

Each provider has an adapter that translates Joch canonical requests into the provider's request format and parses responses into canonical events. Initial adapters:

```text
openai           OpenAI Chat Completions and Responses APIs
anthropic        Anthropic Messages API
google           Vertex AI / Gemini API
microsoft        Azure OpenAI / Foundry
ollama           Ollama HTTP
vllm             vLLM HTTP
llama-cpp        llama.cpp server
generic-openai   any OpenAI-compatible endpoint
```

Adapters are independent packages. Adding a provider does not touch the router core.

## API surface

```text
POST   /v1/model/respond            non-streaming model call
POST   /v1/model/stream             streaming model call
GET    /v1/models/capabilities      capability vector for matching
GET    /v1/route/{name}/explain     explain a routing decision after the fact
```

## Operator commands

```bash
joch top models
joch describe modelroute research-default
joch route exec-123 --explain
joch models check --agent researcher --target anthropic:claude-sonnet
```

## What the router does *not* do

- It does not own conversation state. That belongs to [`Conversation`](../specs/kubernetes/conversation.md).
- It does not implement provider-specific retry semantics that bypass policy. Policy and budget gates always run.
- It does not silently downgrade modality or capability. Capability mismatch is a hard fall-forward decision.

The router is the single place where every provider call is governed, attributed, and observable — without forcing agents to learn a Joch-specific API.
