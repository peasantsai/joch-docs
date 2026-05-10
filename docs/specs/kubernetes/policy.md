# Policy

A `Policy` is a portable, versioned set of rules that the [policy engine](../../architecture/policy-engine.md) enforces at every gateway boundary. The same policy resource works across OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, and custom agents.

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: external-send-requires-approval
  namespace: support-platform
spec:
  description: >
    All external_write tool calls in production require human approval.
    PII in tool arguments is redacted before the call leaves the gateway.

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
        approvalRouting:
          approvers:
            - team:support-platform-leads
          timeoutMinutes: 30
          onTimeout: deny

    - when:
        data.contains: pii
      action:
        redact: true
        redactionRules:
          - emails
          - phoneNumbers
          - ssn

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

  audit:
    logRequests: true
    logDecisions: true
    redactSecrets: true

status:
  phase: Ready
  observedGeneration: 3
  lastEvaluatedAt: "2026-05-10T10:35:03Z"
  evaluations7d: 18243
  denials7d: 12
  modifications7d: 207
```

## Where policies are evaluated

Policies are evaluated at the boundaries every framework crosses:

```text
Tool gateway       on toolCallRequest, toolCallResult
MCP gateway        on protocols/MCP outbound and inbound
Model router       on agentTrigger, message
Memory             on memoryContextRetrieval, memoryStore
RAG                on knowledgeRetrieval
A2A broker         on inter-agent message hooks
```

This boundary enforcement is what makes one `Policy` resource portable across SDKs. See [Hooks](../../aos/hooks.md).

## Selectors

`appliesTo` selects which agents the policy governs:

```yaml
appliesTo:
  agents:
    selector:
      matchLabels:
        env: prod
        team: support-platform
```

Selectors compose. An agent inherits every policy whose selector matches its labels.

## Decision shape

The policy engine returns:

```json
{
  "decision": "allow | deny | modify",
  "reason": "string",
  "modifiedRequest": { "...": "optional" },
  "policyId": "external-send-requires-approval",
  "policyVersion": "v3"
}
```

The decision is recorded as a `HookDecision` event in the [Trace](trace.md) and reflected on the corresponding [`ToolCall`](toolcall.md) or [`Approval`](approval.md).

## Compilation targets

Policies are authored in Joch's portable form. The engine compiles them for runtime evaluation:

- Native interpreter (default).
- OPA / Rego target for deployments that already run Open Policy Agent.
- Cedar target for AWS-style authorization use cases.

The source of truth is always the Joch resource.

## Authoring workflow

```bash
joch validate -f policies/
joch policy preview -f policies/external-send-requires-approval.yaml
joch apply -f policies/ --context staging
joch promote policy/external-send-requires-approval --from staging --to prod
```

`policy preview` replays recent traffic against the new rule set and shows what would change. This is the safest way to ship a policy update.

## Bypass and overrides

Policies cannot be silently bypassed. Two formal escape hatches, both auditable:

- **Break-glass approval** — a designated approver can override a denial on a specific call.
- **Emergency policy disable** — a top-level admin can disable a specific rule for a bounded window.

[Back to the catalog](index.md)
