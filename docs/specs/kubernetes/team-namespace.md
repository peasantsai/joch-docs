# Team / Namespace

A `Team` is the multi-tenant ownership boundary in Joch. Each team owns one or more namespaces; roles bind operators to teams.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Team
metadata:
  name: support-platform
spec:
  description: Owns the support automation agents.

  members:
    - user: alice@example.com
      role: admin
    - user: bob@example.com
      role: operator
    - user: carol@example.com
      role: viewer

  namespaces:
    - support-platform
    - support-platform-staging

  defaultPolicies:
    - external-send-requires-approval
    - no-customer-data-exfiltration

  budgetRefs:
    - support-platform-monthly

status:
  phase: Ready
  memberCount: 3
```

## Roles

```text
admin       full read / write on team namespaces and policies
operator    read / write on agents, deployments, executions, approvals
viewer      read-only access
approver    can approve human-in-the-loop tool calls and promotions
auditor     read-only access plus access to audit log
```

Roles can be extended with custom verbs as the platform evolves.

## Namespaces

Namespaces are the resource-scoping primitive. They make CLI and API addressing predictable: `joch get agents -n support-platform`. RBAC lives at the team level and is propagated to namespaces.

[Back to the catalog](index.md)
