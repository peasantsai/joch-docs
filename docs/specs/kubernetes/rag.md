# RAG

Kubernetes-style YAML resource specification for Joch `RAG` resources.

[Back to Kubernetes specs](index.md)

## RAG spec

`RAG` should define retrieval behavior, not just vector storage.

```yaml
apiVersion: joch.dev/v1alpha1
kind: RAG
metadata:
  name: company-docs-rag
spec:
  sources:
    - name: engineering-docs
      kind: KnowledgeSource
    - name: product-docs
      kind: KnowledgeSource

  embeddingModelRef:
    name: text-embedding-large

  vectorStore:
    type: pgvector
    connectionSecretRef:
      name: rag-postgres
    collection: company_docs

  chunking:
    strategy: semantic
    maxTokens: 800
    overlapTokens: 120

  retrieval:
    topK: 12
    minScore: 0.72
    hybridSearch: true
    reranker:
      enabled: true
      modelRef:
        name: reranker-v1

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

status:
  phase: Ready
  documentsIndexed: 18342
  lastIndexedAt: "2026-05-09T09:45:00Z"
```

Separate `RAG` from `Memory`:

```text
Memory = agent/user/team state
RAG = retrieval over external knowledge corpora
```

---
