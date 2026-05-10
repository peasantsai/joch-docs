# Deployment

Joch AI must run locally, in Docker, and on Kubernetes.

## Local

```bash
joch up
```

Uses:

```text
SQLite
local process gateway
local file storage
```

Local mode is for individual developers and demos.

## Docker Compose

```bash
docker compose up
```

Uses:

```text
Postgres
Joch Server
Joch Gateway
Joch Worker
optional Redis for queueing
```

Compose mode is for team trials and repeatable examples.

## Kubernetes

```bash
helm install joch peasantsai/joch
```

Installs:

```text
Joch Server
Joch Gateway
Joch Worker
Postgres or external DB config
ServiceAccounts
NetworkPolicies
Helm values
optional CRDs later
```

Kubernetes mode is for production and enterprise evaluation.

## Deployment rule

Deployment should not change the resource model. The same `Agent`, `Tool`, `MCPServer`, `Policy`, `ToolCall`, `Approval`, `Execution`, `TraceEvent`, and `ABOM` records must work in every mode.
