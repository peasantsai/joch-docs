# Portability

> Run the same agent record locally, in Docker, on Kubernetes, or with managed runtimes — and migrate conversations across providers without losing tools, memory, or artifacts.

Portability is what makes Joch survive a change of SDK, vendor, or runtime. Provider lock-in is the largest hidden cost in agent operations: a team that ships only on OpenAI cannot react to a price change, an outage, or a new capability somewhere else without rewriting agent state from scratch.

## Two dimensions of portability

```text
Runtime portability     same agent runs locally / Docker / Kubernetes / managed runtime
Provider portability    same conversation can switch between OpenAI / Anthropic / Google / local models
```

The two dimensions are independent and compose: an agent built with Claude Agent SDK can run as a local process today, in Kubernetes tomorrow, and migrate to OpenAI mid-conversation if a [`ModelRoute`](../specs/kubernetes/model-route.md) policy says so.

## Runtime portability

Joch defines a single agent record and selects a runtime via context, not by changing the spec.

```bash
joch context use local
joch apply -f agents/

joch context use prod-k8s
joch apply -f agents/
```

Joch supplies pluggable runtime adapters:

```text
LocalRuntimeAdapter        spawn worker subprocesses
DockerRuntimeAdapter       create Docker containers
KubernetesRuntimeAdapter   create Jobs / Deployments / StatefulSets
NomadRuntimeAdapter        create Nomad job groups
ServerlessRuntimeAdapter   invoke serverless functions
```

The adapters are documented in [Service Architecture](../architecture/service-architecture.md). The agent record never has to know which runtime it is on.

## Provider portability

Joch owns conversation state, not the model provider. The canonical message format includes:

```yaml
conversation:
  apiVersion: joch.dev/v1alpha1
  kind: Conversation
  spec:
    portability:
      canonicalFormat: joch.dev/conversation.v1
      preserveVendorMetadata: true
  events:
    - role: user
      content:
        - kind: text
          text: "Research agent fleet management"
    - role: assistant
      content:
        - kind: text
          text: "Here are the key findings..."
      tool_calls:
        - id: call_1
          name: web.search
          args: { query: "agent fleet management" }
      vendorMetadata:
        openai:
          response_id: resp_abc
          model: gpt-5.5-thinking
```

Provider adapters render this state into each vendor's message format. Adapters never become the source of truth.

## The mid-conversation switch

A switch from one provider to another is not a raw transcript copy. Joch creates a deterministic [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md):

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata:
  name: chk-456
spec:
  conversationRef: { name: conv-123 }
  fromBackend: openai:gpt-5-thinking
  toBackend: anthropic:claude-sonnet
  summary:
    userGoal: "Design state persistence"
    currentTask: "Explain migration strategy"
    decisions:
      - "Use vendor-neutral state as source of truth"
      - "Persist tool calls as event log"
      - "Avoid replaying side-effectful tools"
    openQuestions:
      - "How much vendor metadata to retain"
    activeArtifacts:
      - artifact://design/state-persistence-draft
    relevantMemoryRefs:
      - mem://project/joch/state-model
```

The new provider receives **system + checkpoint + recent window + relevant memory + tool registry** — not the full raw history.

## Capability-aware switching

Not every switch is safe. Joch validates target capabilities before committing:

```bash
joch agents switch researcher \
  --from openai:gpt-5-thinking \
  --to anthropic:claude-sonnet \
  --conversation conv-123 \
  --dry-run
```

```text
Compatibility check:
✓ Text conversation supported
✓ MCP tools supported
✓ 200k context sufficient
⚠ JSON-schema strict mode differs (will use loose-parse fallback)
⚠ Previous turn used image input; target supports images, format will be transformed
✗ Computer-use tool unavailable; tool will be removed for this conversation
```

Policy decides whether warnings or failures gate the switch. See [State Portability](../architecture/state-portability.md) and [Model Router](../architecture/model-router.md).

## Tool-call safety during migration

Tool calls are the largest hazard in mid-conversation switches. Joch persists every [`ToolCall`](../specs/kubernetes/toolcall.md) with side-effect classification and idempotency key. After a switch, the new provider receives a compact summary of past calls:

```text
Earlier in this run, github.create_issue was called with
  { repo: "acme/joch", title: "Add Memory spec" }
producing
  { issueUrl: "https://github.com/acme/joch/issues/123" }.
The action is non-idempotent and must not be repeated.
```

Side-effecting tools are never silently replayed. If a side effect must repeat, an explicit policy or operator action is required.

## ModelRoute

Provider portability is automated by a [`ModelRoute`](../specs/kubernetes/model-route.md):

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

Any agent that uses the route gets capability-checked, cost-aware, and region-aware fallback for free.

## Acceptance criteria

A team operating Joch's Portability pillar can:

- run the same agent record locally, in Docker, and on Kubernetes by changing a context — not the spec,
- migrate a live conversation from OpenAI to Anthropic with a checkpoint and a capability check, with no duplicate side effects,
- pin an agent to a `ModelRoute` and let Joch fall back across providers when the primary fails or exceeds budget,
- export a portable conversation log and replay it against a different provider for debugging.
