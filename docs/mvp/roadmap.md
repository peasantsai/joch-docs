# Roadmap

## Phase 0 - Foundation

Deliver:

```text
brand identity
repo setup
resource schemas
CLI skeleton
server skeleton
SQLite/Postgres store
basic docs
```

Success:

```bash
joch init
joch up
joch get agents
```

## Phase 1 - Registry

Deliver:

```text
Agent registry
Tool registry
MCP registry
manual registration
basic discovery
ABOM v0
```

Success:

```bash
joch discover ./examples
joch abom support-triage
```

## Phase 2 - Gateway

Deliver:

```text
MCP proxy
tool-call logging
tool schema capture
side-effect classification
policy checks
```

Success:

```text
agent calls tool through Joch Gateway
Joch records tool call
policy can allow or deny
```

## Phase 3 - Governance

Deliver:

```text
Policy v0
Approval workflow
Audit log
Risk flags
Tool allow/deny
```

Success:

```text
external write pauses for approval
approval resumes tool call
audit record is complete
```

## Phase 4 - Framework adapters

Deliver:

```text
OpenAI adapter
Claude adapter
LangGraph adapter
CrewAI adapter
generic MCP mode
```

Success:

```text
same policy works across two frameworks
same ABOM format works across two frameworks
```

## Phase 5 - Deployment

Deliver:

```text
Docker Compose
Helm chart
Kubernetes deployment
basic UI optional
```

Success:

```text
team can run Joch in local, Docker, or Kubernetes
```
