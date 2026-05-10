# Business Model

## Initial customer profile

Primary ICP:

```text
AI platform teams
developer platform teams
security-conscious engineering teams
enterprise automation teams
companies adopting multiple agent frameworks
```

Early adopters:

```text
teams running LangGraph, CrewAI, OpenAI, or Claude agents in production
teams adopting MCP servers
teams exposing internal tools to agents
teams needing approval before side effects
teams needing tool-call auditability
```

## Buyer personas

```text
Head of AI Platform
VP Engineering
Developer Platform Lead
Security Engineering Lead
Cloud/Infrastructure Lead
AI Governance Lead
```

## Pain points

Joch AI sells into these problems:

```text
We do not know how many agents exist.
We do not know which tools agents can call.
We do not know which agents can access customer data.
We do not know what MCP servers are running.
We cannot enforce one policy across frameworks.
We cannot audit every tool call.
We cannot require approval before external writes.
We cannot see agent cost by team, tool, model, or workflow.
We cannot produce compliance evidence for agent activity.
```

## Packaging

| Tier | Includes |
|---|---|
| Open-source core | local mode, inventory, basic MCP gateway, basic policy engine, basic trace logs, Docker Compose, Helm install. |
| Team / Pro | multi-user console, team namespaces, shared policy packs, approval workflows, cost dashboards, ABOM exports, advanced retention, hosted registry. |
| Enterprise | SSO/SAML/OIDC, RBAC, audit exports, compliance reports, private deployment, custom policies, Vault integration, SIEM integration, support, air-gapped install, HA. |
| Cloud | managed control plane, self-hosted runtime plane, hosted dashboard, hosted policy registry, billing and usage analytics. |

## Go-to-market wedge

Landing message:

```text
Your agents are calling tools. Who is watching?

Joch AI gives teams one place to discover agents, govern MCP tools,
enforce policies, approve risky actions, and audit every tool call.
```

Developer hook:

```bash
joch mcp scan
joch tools ls
joch policy apply -f policies/no-external-write.yaml
joch gateway start
```

## Moat

The long-term defensibility is not code alone. It is the historical system of record:

```text
cross-framework inventory graph
MCP/tool-call gateway adoption
policy-as-code for agent action
Agent Bill of Materials data
historical execution and tool-call records
risk scoring and compliance workflows
framework/provider adapters
deployment and runtime integrations
open specs adopted by teams
registry of trusted agent components and policies
```
