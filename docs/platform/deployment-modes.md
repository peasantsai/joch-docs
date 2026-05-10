# Deployment Modes

Kubernetes should not be mandatory for every user.

The likely audience includes:

```text
Solo developers
Local-first hackers
Small teams
CI/CD users
Enterprise platform teams
```

Joch should support two primary modes.

## Local Mode

Local mode should feel lightweight:

```bash
joch up
joch apply -f agent.yaml
joch run research-agent
```

Under the hood, it can use:

```text
SQLite/Postgres
Local worker process
Local MCP servers
Docker optional
```

This mode is for development, demos, and quick iteration.

## Cluster Mode

Cluster mode should install the production control plane:

```bash
joch install --cluster
joch apply -f agent.yaml
joch deploy research-agent --env prod
```

Under the hood, it should use:

```text
Kubernetes CRDs
Controllers
Jobs
Deployments
Secrets
Services
NetworkPolicies
OpenTelemetry
```

This mode is for production.

## Familiar Pattern

The model mirrors a pattern developers already understand:

```text
Docker Compose locally
Kubernetes in production
```

For Joch:

```text
Joch local for development
Joch on Kubernetes for production
```

The same resource specs should work in both modes.
