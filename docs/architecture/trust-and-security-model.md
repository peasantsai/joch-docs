# Trust and Security Model

Joch sits in the path of every external action an agent takes. That means it must be carefully bounded: which services see secrets, which services can talk to provider APIs, which services can store data, and which services can be exposed to which tenants.

## Trust zones

```text
┌───────────────────────────────────────────────────────────────┐
│                      Operator zone                            │
│  CLI, web console, CI/CD, automation                          │
└───────────────┬───────────────────────────────┬───────────────┘
                │ apply / read state             │ approve / reject
                ▼                                ▼
┌───────────────────────────────────────────────────────────────┐
│                      Control plane zone                       │
│  apiserver, store, admission, policy engine, scheduler,       │
│  approval service, budget service, compiler                   │
└───────────────┬───────────────────────────────┬───────────────┘
                │ desired state                  │ hooks (allow/deny/modify)
                ▼                                ▲
┌───────────────────────────────────────────────────────────────┐
│                      Data plane zone                          │
│  tool gateway, mcp gateway, model router,                     │
│  memory, rag, artifact, trace, a2a-broker                     │
└───────────────┬───────────────────────────────┬───────────────┘
                │ resolved secrets               │ traces, costs
                ▼                                ▲
┌───────────────────────────────────────────────────────────────┐
│                      External zone                            │
│  provider APIs, MCP servers, external SaaS, customer DBs      │
└───────────────────────────────────────────────────────────────┘
```

Each zone has narrower trust than the one below and is constrained by network policies (in Kubernetes mode) or process-level isolation (in Docker / local mode).

## Secret handling

- Secret **values** are never stored in Joch records. `Secret` resources hold references only (Vault path, Kubernetes secret name, AWS Secrets Manager ARN, env var name).
- Secret resolution happens at the data-plane gateway boundary: the secret broker injects the value just before the outbound call and never returns it to the control plane or the agent.
- Logs and traces redact known secret-shaped values by default. Operators can register additional patterns per environment.

## Network egress

- Tool gateway, MCP gateway, and model router are the **only** services with allow-listed egress. Workers, memory, RAG, artifact, and trace cannot reach the public internet directly.
- Egress allow-lists are policy-driven. A new external destination requires a `Policy` change, not a code change.
- In Kubernetes mode, NetworkPolicies make this enforceable at the cluster network layer.

## Multi-tenancy boundary

- `Team / Namespace` is the primary ownership boundary. RBAC binds operators to namespaces.
- `Environment` is the promotion boundary. A staging operator cannot apply to prod without explicit role.
- For SaaS deployments, `Tenant` wraps namespaces. Cross-tenant data must not exist; the resource store and the trace store are tenant-keyed at the row level.

## Prompt-injection mitigations

- Inbound MCP results are scanned for known injection patterns; matches can be denied, modified, or annotated per policy.
- Tool results that arrive from third-party services are likewise scanned at the tool gateway.
- The Guardian Agent role can be doubled — a security-team-owned Guardian can run alongside the platform-owned policy engine for separation of duties.

## Authentication

- Operator authentication: OIDC, with optional Kubernetes service account integration.
- Agent authentication: Joch issues short-lived workload identities to runtime workers; tool and MCP calls carry the identity for audit.
- Provider authentication: handled by the model router via the secret broker; provider keys never leave the data plane.

## Audit log

Every API call, every apply, every promote, every approval, every override is an immutable audit record. Audit records are exportable to OCSF for ingest by SIEMs. See [OCSF Mapping](../aos/ocsf-mapping.md).

## Threat model summary

| Threat | Mitigation |
|---|---|
| Untrusted MCP server | Registry, version pinning, sandboxing, schema-drift detection, trust scoring, scanning. |
| Prompt injection in tool result | Inbound scanning + Guardian `modify` / `deny`. |
| Compromised SDK code | Tools and MCP go through gateways; SDK cannot bypass policy or hide cost. |
| Stolen provider API key | Keys live in secret broker; rotation is one operator command. |
| Cross-tenant leakage | Tenant-keyed storage; RBAC at API layer; NetworkPolicies in cluster mode. |
| Silent regression | Eval gates, ABOM diffs, structural diffs at promote. |
| Audit tampering | Immutable audit store; hash chains for high-trust deployments. |
| Provider outage | ModelRoute fallback; capability and cost validation prevent unsafe degradation. |
| Cost runaway | Budget service enforces per-run / per-day caps; alerts on anomaly. |
| Insider override abuse | Break-glass overrides require separate role; emergency policy disable is bounded and auto-reverts. |

## What this model does *not* claim

- Joch is not a TPM or HSM. It uses the secret broker abstraction; high-assurance environments back the broker with HSM-stored keys.
- Joch does not eliminate model-side risks (jailbreaks, hallucinations). It surfaces them via traces and lets the Guardian Agent intervene at the gateway boundary.
- Joch does not replace the SDK's own input/output guardrails. Those run inside the SDK; Joch governs the **outbound** boundary.

The trust model is conservative by design: sensitive operations live behind narrow, auditable APIs, and the gateway boundary is the single point where SDK code meets the rest of the world.
