# Approvals

> A support agent occasionally needs to send a financial refund. The action requires human approval, but the agent must not block other ticket triage while waiting.

## Symptoms

- Side-effecting tool calls run unsupervised and an irreversible action surprises the team.
- Approvals are buried in chat, with no audit trail.
- A single pending approval blocks all of the agent's other work.
- The same approver becomes a bottleneck across many agents.

## What Joch does

The [Tool Gateway](../architecture/tool-gateway.md) classifies side effects, the [Policy Engine](../architecture/policy-engine.md) decides which calls require approval, and the approval service routes [`Approval`](../specs/kubernetes/approval.md) records to the right humans without blocking the agent loop.

## Walkthrough

### 1. Classify the tool

```yaml
apiVersion: joch.dev/v1alpha1
kind: Tool
metadata: { name: refund.issue }
spec:
  type: rest
  endpoint: { url: "https://billing.example.com/refunds", method: POST }
  sideEffects: { level: financial, idempotent: false, requiresApproval: true }
```

### 2. Policy that requires approval

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata: { name: financial-actions-require-approval }
spec:
  appliesTo: { agents: { selector: { matchLabels: { env: prod } } } }
  rules:
    - when: { tool.sideEffect: financial }
      require:
        approval: human
        approvalRouting:
          approvers: [team:billing-leads]
          timeoutMinutes: 60
          onTimeout: deny
        channels:
          - slack:#billing-approvals
          - email:billing-leads@example.com
```

### 3. Non-blocking approval

When the agent calls `refund.issue`, the gateway:

1. Creates an [`Approval`](../specs/kubernetes/approval.md) record.
2. Sends the approval to Slack and email.
3. Suspends only the originating tool call — the agent can continue with other work where the SDK allows.
4. Resumes the call when an approver decides, or denies on timeout.

### 4. Approver experience

```bash
joch approvals ls
joch describe approval approval-call-abc123
joch approvals decide approval-call-abc123 --decision approve --rationale "Refund eligibility confirmed."
```

A web console and Slack / email integrations give the same actions. Decisions are immutable and feed the audit log.

### 5. Break-glass override

```bash
joch approvals override approval-call-abc123 --decision approve --grant-by sre-oncall \
  --reason "Outage: payment system down; refunds queued; manually granting."
```

Overrides are bounded, logged, and require a separate role — they cannot be invoked from within the agent or its SDK.

## Resources involved

- [`Approval`](../specs/kubernetes/approval.md)
- [`ToolCall`](../specs/kubernetes/toolcall.md), [`Tool`](../specs/kubernetes/tool.md)
- [`Policy`](../specs/kubernetes/policy.md)

## Outcome

Risky actions are auditable, routed to the right humans, and bounded by timeout. Approvers operate from a single queue across all agents. Agents do not stall the rest of their work waiting for one decision.
