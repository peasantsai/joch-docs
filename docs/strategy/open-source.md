# Open Source Strategy

## License

Recommended default:

```text
Apache-2.0
```

Apache-2.0 is adoption-friendly for infrastructure software and easier for enterprises to approve.

An alternative is AGPL for the server and Apache/MIT for SDKs. That may improve business defensibility but increases adoption friction.

## Open-source core

The open-source core should be real infrastructure, not a crippled trial:

```text
CLI
server
gateway
basic policies
basic registry
basic ABOM
Docker deployment
Helm chart
framework adapters
```

## Commercial features

Commercial packaging should focus on operations at scale:

```text
SSO/SAML
RBAC
advanced audit exports
policy packs
hosted dashboard
multi-tenant cloud
long-term trace retention
SIEM integrations
enterprise support
approval workflows at scale
compliance reporting
```

## Boundary rule

The open-source product must be sufficient to prove the wedge:

```text
discover/register agents
inventory tools and MCP servers
proxy and audit tool calls
enforce policies
require human approval
generate an ABOM
run locally, in Docker, and on Kubernetes
integrate without agent rewrites
```
