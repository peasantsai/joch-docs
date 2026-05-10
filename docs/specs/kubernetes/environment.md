# Environment

An `Environment` is a promotion boundary — typically `dev`, `staging`, or `prod` — with bound policies, budgets, defaults, and runtime configuration. Environments make `joch promote` predictable and auditable.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Environment
metadata:
  name: prod
spec:
  description: Production environment for the support platform.

  region: eu-central

  runtime:
    orchestrator: kubernetes
    cluster: prod-agent-cluster

  defaults:
    modelRouteRef:
      name: research-default
    traceRetentionDays: 30
    logLevel: info

  compliance:
    dataResidency: EU
    piiMode: redact
    auditRequired: true

  policies:
    - name: external-send-requires-approval
    - name: no-customer-data-exfiltration

  budgetRefs:
    - name: support-platform-monthly

status:
  phase: Ready
```

## Why a separate kind

Without `Environment`, every agent and deployment would have to repeat the same defaults, policies, and budgets. With `Environment`, those settings are declared once and applied to every record promoted into the environment.

## Promotion

```bash
joch promote agent/support-triage --from staging --to prod
joch promote deployment/support-triage-prod --from staging --to prod
joch promote policy/external-send-requires-approval --from staging --to prod
```

A failed eval or missing approval blocks the promotion with a clear diagnostic.

[Back to the catalog](index.md)
