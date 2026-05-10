# Custom Code Integration

For agents that do not use a published SDK, Joch provides `custom-python` and `custom-typescript` framework adapters. The contract is intentionally minimal: an entrypoint plus access to the Joch runtime client.

## Python

### Contract

```python
# my_agent/entrypoint.py
from joch.runtime import JochRuntime

async def joch_entrypoint(execution_context: dict, joch: JochRuntime) -> dict:
    """
    execution_context — the Execution record (input, agentRef, traceId, ...)
    joch              — runtime client wired to the Joch tool gateway,
                         MCP gateway, model router, memory, RAG, trace, etc.
    """
    user_input = execution_context["input"]["content"]

    # Plan / loop in your own code
    research = await joch.tools.call("web.search", {"query": user_input[0]["text"]})
    draft    = await joch.model.respond(
        route="research-default",
        messages=[{"role": "user", "content": user_input}],
    )

    artifact_uri = await joch.artifact.write(
        name="report.md",
        content=draft.text,
        content_type="text/markdown",
    )
    return {"artifactRef": artifact_uri}
```

The runtime client gives you:

```text
joch.tools.call(name, input)              # routes through Tool Gateway with AOS hooks
joch.mcp.call(server, method, params)     # routes through MCP Gateway
joch.model.respond(...)                   # routes through Model Router with ModelRoute
joch.model.stream(...)                    # streamed model call
joch.memory.read(...)                     # AOS memoryContextRetrieval
joch.memory.write(...)                    # AOS memoryStore
joch.rag.retrieve(...)                    # AOS knowledgeRetrieval
joch.artifact.write(...)                  # writes Artifact record
joch.trace.event(name, payload)           # custom span / event
joch.handoff.to(other_agent, summary)     # creates a Handoff record
joch.approval.request(...)                # requires_approval flow
```

Every call carries the `traceId`, `executionId`, agent identity, and tenant context; you do not pass them.

### Agent record

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-bespoke
spec:
  framework:
    adapterRef: { name: custom-python }
    entrypoint: ./my_agent/entrypoint.py
    pythonModule: my_agent.entrypoint:joch_entrypoint
    image: ghcr.io/example/research-bespoke:1.0.0
  modelRoute: { routeRef: { name: research-default } }
  tools:
    - name: web.search
  policies:
    - name: research-cost-cap
```

## TypeScript

### Contract

```typescript
// src/entrypoint.ts
import type { ExecutionContext, ExecutionResult, JochRuntime } from "@joch/runtime";

export async function jochEntrypoint(
  ctx: ExecutionContext,
  joch: JochRuntime,
): Promise<ExecutionResult> {
  const input = ctx.input.content;

  const research = await joch.tools.call("web.search", { query: input[0].text });
  const draft = await joch.model.respond({
    route: "research-default",
    messages: [{ role: "user", content: input }],
  });

  const artifactUri = await joch.artifact.write({
    name: "report.md",
    content: draft.text,
    contentType: "text/markdown",
  });

  return { artifactRef: artifactUri };
}
```

### Agent record

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-bespoke
spec:
  framework:
    adapterRef: { name: custom-typescript }
    entrypoint: ./src/entrypoint.ts
    typescriptModule: dist/entrypoint:jochEntrypoint
    image: ghcr.io/example/research-bespoke:1.0.0
  modelRoute: { routeRef: { name: research-default } }
```

## When to use custom code

- An organization with strong Python or TypeScript tooling that prefers minimum SDK lock-in.
- An agent loop that does not fit any published SDK's shape (e.g., custom planner, simulation loop, novel orchestration).
- A glue layer that combines two SDKs in non-trivial ways.

For most teams, one of the SDK-specific framework adapters is the right answer. The custom adapter is the escape hatch that ensures Joch is never a barrier to building the agent you actually need.

## What Joch still gives you

- Tool gateway with AOS hooks and idempotency.
- MCP gateway with version pinning and scanning.
- Model router with `ModelRoute` fallback and budgets.
- AgBOM emission for the bespoke agent.
- Trace, approvals, evals, release gates.

You do not lose governance by going custom. You only give up the SDK-specific ergonomics on the agent side.
