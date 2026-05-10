# RAG

A `RAG` resource describes a retrieval index used by an agent. It composes one or more [`KnowledgeSource`](knowledge-source.md) resources and routes retrievals through the [RAG data-plane service](../../architecture/data-plane.md), which exposes the AOS [`knowledgeRetrieval` hook](../../aos/hooks.md).

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: RAG
metadata:
  name: support-docs-rag
  namespace: support-platform
spec:
  description: RAG index over public support docs and internal runbooks.

  sources:
    - name: support-docs-public
      kind: KnowledgeSource
    - name: support-runbooks-internal
      kind: KnowledgeSource

  embeddingModelRef:
    name: text-embedding-3-large

  vectorStore:
    type: pgvector
    connectionSecretRef:
      name: support-pgvector
    collection: support_docs

  chunking:
    strategy: semantic
    maxTokens: 800
    overlapTokens: 120

  retrieval:
    topK: 5
    minScore: 0.45
    hybridSearch: true
    reranker:
      enabled: true
      modelRef:
        name: rerank-3

  citations:
    required: true
    includeSourceUri: true
    includeLineNumbers: true

  freshness:
    syncInterval: 1h
    staleAfter: 7d

  permissions:
    inheritFromSource: true
    enforceAtQueryTime: true

  policies:
    - name: customer-tier-scoping

  observability:
    logRetrievals: true

status:
  phase: Ready
  documentsIndexed: 12483
  chunkCount: 86120
  lastIndexedAt: "2026-05-09T22:00:00Z"
  retrievalCount7d: 4128
  citationRate7d: 0.91
```

## RAG vs. Memory

```text
Memory        agent / user / team state owned by Joch
RAG           retrieval over external knowledge corpora
```

Keep them separate so retention, indexing, and access controls do not collide.

## Hooks

`knowledgeRetrieval` is called after the RAG service returns results and before the agent attaches them to the model context. A Guardian Agent can:

- `deny` retrievals that contain customer-tier data the agent is not entitled to see,
- `modify` results to scope by tenant or user,
- annotate results with provenance metadata so the trace surfaces citations.

## Quality observability

`status.citationRate7d` and per-execution citation tracking feed the quality metrics in the [Observability pillar](../../pillars/observability.md). Drops in citation rate are an early signal of regression after deployments.

[Back to the catalog](index.md)
