# Memory

Kubernetes-style YAML resource specification for Joch `Memory` resources.

[Back to Kubernetes specs](index.md)

## Memory spec

Memory should be split into different types.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Memory
metadata:
  name: research-agent-working-memory
spec:
  type: working

  scope:
    ownerKind: Agent
    ownerName: research-agent
    namespace: default

  backend:
    type: postgres
    connectionSecretRef:
      name: memory-postgres

  retention:
    ttl: 7d
    maxItems: 10000

  schema:
    fields:
      - name: content
        type: text
      - name: importance
        type: float
      - name: source
        type: string
      - name: expiresAt
        type: timestamp

  access:
    readableBy:
      - research-agent
      - reviewer-agent
    writableBy:
      - research-agent

  privacy:
    piiHandling: redact
    encryption: required

status:
  phase: Ready
  itemCount: 532
```

Memory types I would support:

```text
working      short-lived task context
episodic     history of past runs/events
semantic     durable facts and concepts
procedural   learned workflows/playbooks
preference   user/team preferences
blackboard   shared multi-agent workspace
```

Do not make all memory vector-based. Some memory should be relational, some document-based, some key-value, and some vector-searchable.

---
