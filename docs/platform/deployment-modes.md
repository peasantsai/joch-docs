# Deployment Modes

Joch supports four deployment modes. The same resource specs work across all of them — only the runtime adapter changes.

## Local

For developers. Single process, SQLite store, local files, no Kubernetes.

```bash
brew install joch
joch up
joch apply -f examples/support-triage.yaml
joch run support-triage "Triage incoming queue"
```

Best for fast iteration, prototyping, and example agents.

## Docker Compose

For small teams and self-hosted deployments. Postgres store, Redis event bus, MinIO artifacts, OpenTelemetry collector.

```bash
joch init docker-compose
docker compose up -d
joch context use docker
joch apply -f .
```

Best for team dev environments, CI smoke tests, and demos.

## Kubernetes

For production. Native CRDs, Helm chart, controllers, NetworkPolicies, ServiceMonitor, autoscaling.

```bash
helm install joch joch/joch -n joch-system --create-namespace
kubectl apply -f .
```

Best for multi-tenant, scale-out, governed production fleets.

## Managed SaaS + customer runtime

For organizations that want Joch's UI, traces, and ABOM hosted, but tools, memory, and provider calls inside the customer environment.

```text
joch Cloud control plane (multi-tenant)
        │ secure tunnel / API
        ▼
customer runtime plane (Kubernetes / Docker / VM)
```

Best for regulated industries, EU residency, and teams that do not want to operate the control plane.

## Familiar pattern

The model mirrors the pattern engineering teams already understand:

```text
Local                Docker Compose         Kubernetes / SaaS
prototyping          team dev / CI          production
```

For Joch:

```text
joch local           joch docker            joch on Kubernetes / Joch Cloud
```

The same `joch apply -f .` and the same resource specs work in each mode.
