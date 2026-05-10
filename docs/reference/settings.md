# Settings

Joch separates **configuration** (per-service, runtime) from **settings** (org-level, tenant-level, environment-level). Settings live as resources and are versioned, applied, and audited like any other resource.

## Org settings

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: OrgSettings
metadata: { name: default }
spec:
  defaultEnvironment: dev
  defaultModelRouteRef: { name: research-default }
  agbom:
    formats: [cyclonedx, spdx]
    signing:
      enabled: true
      keyRef: { name: joch-agbom-signing-key }
  trace:
    retentionDays: 30
    sampling: tail
  approval:
    defaultTimeoutMinutes: 30
  policy:
    audit:
      logRequests: true
      logDecisions: true
      redactSecrets: true
```

## Environment settings

Bound to an [`Environment`](../specs/kubernetes/environment.md):

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: Environment
metadata: { name: prod }
spec:
  region: eu-central
  runtime:
    orchestrator: kubernetes
    cluster: prod-agent-cluster
  defaults:
    modelRouteRef: { name: research-default }
    traceRetentionDays: 90
    logLevel: info
  compliance:
    dataResidency: EU
    piiMode: redact
    auditRequired: true
  policies:
    - name: external-send-requires-approval
    - name: no-customer-data-exfiltration
```

## Feature flags

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: FeatureFlags
metadata: { name: default }
spec:
  flags:
    a2a-broker: stable
    abom-spdx: stable
    abom-swid: beta
    eval-llm-judge: stable
    realtime-adapter: experimental
```

Feature flags are explicit, versioned, and respected by all services. There are no hidden environment-variable feature flags in production code paths.

## Admission defaults

Admission applies defaults to incoming records. Override with environment-level settings:

| Resource | Field | Default |
|---|---|---|
| `Agent` | `spec.observability.tracing` | `enabled` |
| `Agent` | `spec.observability.agbom` | `enabled` |
| `Trace` | `spec.sampling.mode` | env-dependent |
| `Trace` | `spec.retention.days` | 30 |
| `Budget` | `spec.enforcement.softLimitPct` | 0.8 |
| `Approval` | `spec.routing.timeoutMinutes` | 30 |
| `Policy` | `spec.audit.logRequests` | `true` |
| `MCPServer` | `spec.security.pinServerVersion` | `true` |

## Roles

Default roles. Bound to teams via `Team` records:

| Role | Verbs |
|---|---|
| `admin` | full read/write on team namespaces and policies |
| `operator` | read/write on agents, deployments, executions, approvals |
| `viewer` | read-only |
| `approver` | decide approvals (tool calls + promotions) |
| `auditor` | read-only + audit log access |

## Notification channels

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: NotificationChannel
metadata: { name: support-platform-leads-slack }
spec:
  type: slack
  channel: "#support-approvals"
  webhookSecretRef: { name: slack-incoming-webhook }
```

```yaml
apiVersion: ops.joch.dev/v1alpha1
kind: NotificationChannel
metadata: { name: ops-pager }
spec:
  type: pagerduty
  serviceSecretRef: { name: pagerduty-key }
```

`Approval.spec.channels` and `Policy` denial alerts reference these channels.

## Self-managed vs managed

In Joch Cloud, org settings, environment settings, and feature flags are managed via the hosted Console; the underlying resources are still YAML and exportable for GitOps.
