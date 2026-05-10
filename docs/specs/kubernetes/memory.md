# Memory

A `Memory` resource binds an agent to a memory store — working, semantic, episodic, procedural, preference, or shared blackboard. Memory operations cross the [memory data-plane service](../../architecture/data-plane.md), where the policy engine can `allow`, `deny`, or `modify` reads and writes per the AOS [`memoryContextRetrieval` and `memoryStore` hooks](../../aos/hooks.md).

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Memory
metadata:
  name: support-triage-working
  namespace: support-platform
spec:
  type: working
  description: Working memory for the support-triage agent (per-execution scratchpad).

  scope:
    ownerKind: Agent
    ownerName: support-triage

  backend:
    type: postgres
    connectionSecretRef:
      name: support-postgres

  retention:
    ttl: 24h
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
      - support-triage
      - support-escalation
    writableBy:
      - support-triage

  privacy:
    piiHandling: redact
    encryption: required

  policies:
    - name: pii-redaction-on-write

status:
  phase: Ready
  itemCount: 412
```

## Memory types

```text
working      short-lived task context, per-execution
episodic     history of past runs and events
semantic     durable facts and concepts
procedural   learned workflows / playbooks
preference   user / team preferences
blackboard   shared multi-agent workspace
```

## Backends

Backends are pluggable adapters. The `Memory` resource is the same shape across them.

```text
postgres        for KV / structured memory
pgvector        for semantic / vector memory
qdrant          dedicated vector store
pinecone        managed vector store
weaviate        managed vector store
redis           hot, ephemeral working memory
```

Not all memory should be vector-based. Joch supports relational, document, key-value, and vector backends so each memory type uses the right shape.

## Hooks

Every read goes through `memoryContextRetrieval`; every write goes through `memoryStore`. A Guardian Agent can:

- redact PII from a working-memory write,
- deny a long-term memory write that contains customer-tier data,
- modify retrieval results to scope per tenant or per user.

[Back to the catalog](index.md)
