# Governance

> Policy-as-code for tool use, data access, approvals, model choice, memory writes, and budgets — enforced at the right boundary, in the same language, across every framework.

Governance is the policy enforcement point. Each SDK has its own ad-hoc guardrails. Joch defines a portable policy resource and enforces it at the gateways every agent crosses, so all agents — regardless of SDK — get the same controls.

## What governance covers

```text
Tool calls           which tools an agent may call, with what arguments, and when an approval is required
Model selection      which providers and models an agent may use, with deny-lists and capability matching
Memory writes        what an agent may persist to working / semantic / episodic memory
Network egress       which destinations a tool may reach
Data classes         PII, PHI, customer-tier data — redaction rules and residency
Cost budgets         per-run, per-day, per-org caps with hard and soft limits
Audit                what is logged, what is redacted, and how long it is retained
```

Each policy is a versioned resource with selectors, rules, and an enforcement boundary. The same resource works across OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, and CrewAI agents.

## Where policy is enforced

Joch enforces policy at the boundaries that every framework crosses:

```text
Agent SDK (any framework)
        │
        ▼
[ Joch Tool Gateway ]    ← portable Policy + AOS hooks (toolCallRequest, toolCallResult)
        │
        ▼
[ Joch MCP Gateway  ]    ← portable Policy + AOS hooks (protocols/MCP outbound + inbound)
        │
        ▼
[ Joch Model Router ]    ← portable Policy + AOS hooks (message)
        │
        ▼
[ Joch Memory       ]    ← portable Policy + AOS hooks (memoryContextRetrieval, memoryStore)
        │
        ▼
[ Joch RAG          ]    ← portable Policy + AOS hooks (knowledgeRetrieval)
```

The gateways are documented in [Tool Gateway](../architecture/tool-gateway.md), [MCP Gateway](../architecture/mcp-gateway.md), [Policy Engine](../architecture/policy-engine.md), and [Model Router](../architecture/model-router.md). Every gateway implements the [AOS hook contract](../aos/hooks.md), returning `allow`, `deny`, or `modify` per OWASP AOS.

## Portable policy example

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: no-external-send-without-approval
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
        model.provider: unknown
      action:
        deny
    - when:
        cost.runUsd: ">5"
      action:
        deny
        reason: budget-exceeded
```

This single policy applies to every prod agent, regardless of framework. There is no per-SDK policy code to maintain.

## Hooks (AOS Instrument)

Joch's gateways implement the OWASP AOS Instrument hooks. A Guardian Agent — Joch's policy engine, or a third-party Guardian — receives every hook call and returns one of three decisions:

| Decision | Behavior |
|---|---|
| `allow` | Continue with the original request. |
| `deny` | Block the request; the agent receives a denial with a reason. |
| `modify` | Continue with `modifiedRequest` (e.g., redacted query, scoped tool argument, summarized context). |

The hook surface includes `agentTrigger`, `toolCallRequest`, `toolCallResult`, `message` (user and agent), `memoryContextRetrieval`, `memoryStore`, `knowledgeRetrieval`, `protocols/MCP` (outbound and inbound), and the A2A hooks. See [Hooks](../aos/hooks.md) for the full taxonomy.

## Approvals

Some side-effecting actions require human review. Joch's [`Approval`](../specs/kubernetes/approval.md) resource models that flow:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Approval
metadata:
  name: approve-call-abc123
spec:
  toolCallRef:
    name: call-abc123
  policyRef:
    name: external-send-requires-approval
  routing:
    approvers:
      - team:support-platform-leads
    timeoutMinutes: 30
    onTimeout: deny
status:
  phase: Pending
```

Approvers act through the CLI, the web console, or webhook integrations (Slack, email, ticketing). Decisions are auditable.

## What governance does *not* try to do

- Joch does not replace each SDK's input/output guardrails. Those continue to run inside the SDK. Joch governs the **outbound boundary** that crosses into tools, models, memory, and the network.
- Joch does not invent new policy languages where existing ones suffice. Policies can compile to OPA/Rego or Cedar where helpful, but the source-of-truth is the Joch `Policy` resource for cross-SDK portability.
- Joch does not silently rewrite agent code. The SDK code is unchanged; the gateway boundary is where enforcement happens.

## Acceptance criteria

A team operating Joch's Governance pillar can:

- write one policy that blocks `email.send` without approval and have it enforced for every prod agent the next time it deploys, regardless of SDK,
- prove a denied call was denied, by which rule, at what time, on which inputs, with the policy version active at the time,
- redact PII from tool arguments before they leave the gateway, with audit evidence,
- promote a policy from staging to prod with a diff and a rollback path.
