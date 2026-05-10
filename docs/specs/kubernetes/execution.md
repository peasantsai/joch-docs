# Execution

An `Execution` is one concrete run of an [`Agent`](agent.md). It owns model calls, tool calls, memory operations, RAG retrievals, traces, costs, and artifacts produced during the run.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Execution
metadata:
  name: exec-20260510-001
  namespace: support-platform
spec:
  agentRef: { name: support-triage }

  trigger:
    type: webhook
    sourceRef:
      name: zendesk-ticket-created

  input:
    content:
      - kind: data
        data:
          ticketId: "T-77123"
          subject: "Refund request for order 12345"

  conversationRef: { name: conv-77123 }

  modelRouteOverride:
    routeRef: { name: research-default }

  budgets:
    inheritFromAgent: true

  observability:
    sampling: full

status:
  phase: Succeeded
  startedAt:   "2026-05-10T10:34:50Z"
  completedAt: "2026-05-10T10:35:14Z"
  costUsd: 0.62
  durationMs: 24000
  modelCalls: 9
  toolCalls: 4
  approvals: 1
  hookDecisions: 24
  artifactRefs:
    - artifact://exec/exec-20260510-001/draft-reply.md
  traceRef: { name: trace-exec-20260510-001 }
  abomSnapshotRef: { name: support-triage-abom, generation: 17 }
```

## Trigger types

```text
manual          operator command (joch run, web console)
webhook         external system event (Zendesk, GitHub, Slack)
schedule        cron-style recurrence
upstream        triggered by another agent's Handoff
```

The trigger maps to an [AOS `agentTrigger` hook](../../aos/hooks.md), so the policy engine can `allow`, `deny`, or `modify` the triggering payload before the execution starts.

## ABOM snapshot

`status.abomSnapshotRef` pins the ABOM generation that was active when the execution started. This makes the execution reproducible: the exact set of tools, MCP servers, models, and policies in effect is recoverable years later.

## Cost and budget

`status.costUsd` is computed from model and tool call costs and reconciled with the [`Budget`](budget.md) controller after completion. A run that exceeds budget is flagged and may be terminated mid-flight if policy permits.

## Replay

Executions are replayable from the trace plus the ABOM snapshot. Replay is gated by policy because it may produce duplicate side effects.

```bash
joch executions replay exec-20260510-001 --policy strict
```

[Back to the catalog](index.md)
