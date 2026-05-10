# Install

The MVP supports three deployment modes: local, Docker Compose, and Kubernetes.

## Local

Local mode is for development and first-day evaluation.

```bash
joch init
joch up
joch get agents
```

Local mode uses:

```text
SQLite
local process gateway
local file storage
```

## Docker Compose

Compose mode is the default team trial.

```bash
docker compose up
```

Compose mode uses:

```text
Postgres
Joch Server
Joch Gateway
Joch Worker
optional Redis for queueing
```

## Kubernetes

Kubernetes is the target production deployment.

```bash
helm install joch peasantsai/joch
```

Kubernetes installs:

```text
Joch Server
Joch Gateway
Joch Worker
Postgres or external database configuration
ServiceAccounts
NetworkPolicies
Helm values
optional CRDs in later phases
```

## Verify

```bash
joch version
joch doctor
joch get agents
joch get mcpservers
```

`joch doctor` should verify API reachability, datastore health, worker status, and gateway readiness.
