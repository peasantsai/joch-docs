# Policy Engine

The policy engine is Joch's portable enforcement layer. It evaluates [`Policy`](../specs/kubernetes/policy.md) records against decision contexts produced by the gateways and returns one of three outcomes — `allow`, `deny`, or `modify` — in alignment with the [OWASP AOS hook contract](../aos/hooks.md).

## What it evaluates

The engine receives decision contexts from every gateway:

```text
Tool Gateway     toolCallRequest, toolCallResult
MCP Gateway      protocols/MCP outbound, protocols/MCP inbound (also toolCall* if a tools/call)
Model Router     message (user), message (agent), agentTrigger
Memory           memoryContextRetrieval, memoryStore
RAG              knowledgeRetrieval
A2A Broker       send_message, stream_message, cancel_request, get_task, ...
```

Each context carries the `agent`, `framework`, `tenant`, `environment`, `tool` (or `model` / `mcpServer`), the request payload, and the AOS step metadata (`turnId`, `stepId`).

## Decision shape

The engine returns:

```json
{
  "decision": "allow | deny | modify",
  "reason": "string",
  "modifiedRequest": { "...": "optional" },
  "policyVersion": "v3",
  "policyId": "no-external-send-without-approval"
}
```

The decision is recorded as a `HookDecision` event in the trace. Every outcome — even `allow` — is auditable.

## Portable policy resource

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: no-external-send-without-approval
  namespace: support-platform
spec:
  appliesTo:
    agents:
      selector:
        matchLabels:
          env: prod
  rules:
    - when:
        tool.sideEffect: external_write
      require:
        approval: human
    - when:
        data.contains: pii
      action:
        redact: true
    - when:
        model.provider: not-allowlisted
      action:
        deny
        reason: model-not-permitted
    - when:
        cost.runUsd: ">5"
      action:
        deny
        reason: budget-exceeded
```

A single `Policy` resource works across SDKs, frameworks, and runtimes. Rules are evaluated in declared order; `deny` short-circuits.

## Compilers

The engine compiles `Policy` resources into the form best for runtime evaluation:

- A native interpreter for inline `when` / `action` clauses.
- An optional compile-to-Rego target for deployments that already run [Open Policy Agent](https://www.openpolicyagent.org/).
- An optional compile-to-Cedar target for AWS-style authorization use cases.

The source of truth remains the Joch resource. Compilation produces an artifact that is referenced from the [CompiledAgentManifest](control-plane.md#compiled-agent-manifests).

## Guardian Agent role

In OWASP AOS terminology, the Guardian Agent is the entity that receives hooks and returns decisions. By default, Joch's policy engine plays the Guardian role for every Joch-managed agent.

Operators can substitute or compose:

- A third-party Guardian Agent can be plugged in per environment or per agent.
- Multiple Guardian Agents can compose: any `deny` is a `deny`; `modify` outputs are merged in declared order; `allow` is the unanimous outcome.

This composition is critical for separation of duties: a security team can run its own Guardian for PII / DLP rules, while a platform team runs the default policy engine for general operations.

## Bypass and emergency overrides

Policies cannot be silently bypassed. There are two formal escape hatches, both auditable:

- **Break-glass approval** — a designated approver can grant a one-shot override on a denied decision. The override is recorded with reason and operator identity.
- **Emergency policy disable** — a top-level admin can disable a specific rule (not a whole policy) for a bounded time window, with mandatory follow-up.

Both flows produce immutable audit events.

## Performance

The engine is on the hot path of every gateway call. Rules are precompiled per `applyTo` selector and indexed in memory. Typical p95 evaluation latency is sub-millisecond. The engine is horizontally scalable; gateways can call it via local socket (sidecar mode) or over the cluster network.

## Authoring policies

Policies are versioned resources. Recommended workflow:

1. Author or update a `Policy` YAML.
2. `joch validate -f policies/` — schema and selector checks.
3. `joch policy preview -f policies/` — replays recent traffic and shows what would change.
4. `joch apply -f policies/` to staging.
5. `joch promote policy ... --from staging --to prod` after review.

## What the policy engine does *not* do

- It does not author prompts. It does not score agent quality.
- It does not own approvals end-to-end; it triggers approvals through the [`Approval`](../specs/kubernetes/approval.md) resource and the approval service.
- It does not re-implement OPA or Cedar; it can target them as compilation backends, but the source of truth is the portable `Policy` resource.

The engine is a small, fast, deterministic decision service. Its value is the breadth of contexts it sees — every gateway in the data plane — and the portability of its rules.
