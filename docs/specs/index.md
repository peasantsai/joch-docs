# Resources

Joch's resource model is a small, opinionated catalog of Kubernetes-style YAML kinds for managing AI agent fleets across SDKs, vendors, and runtimes. The kinds are designed for **operations** — inventory, governance, portability, observability, release management — not for re-implementing what vendor SDKs already do.

## How resources are organized

<div class="grid cards" markdown>

-   :material-shape-outline: **Resource Model**

    ---

    The conceptual model: design philosophy, API groups, relationships, and how desired-state and runtime-state resources fit together.

    [Read the resource model](resource-model.md)

-   :material-format-list-bulleted-square: **Catalog**

    ---

    One Kubernetes-style YAML page per resource kind, with full spec, status, and example.

    [Browse the catalog](kubernetes/index.md)

</div>

## Resource groups

Joch groups resources by lifecycle:

```text
Identity / desired state
  Agent, FrameworkAdapter, Model, ModelRoute, Tool, MCPServer,
  Memory, RAG, KnowledgeSource, Policy

Runtime
  Execution, Conversation, StateCheckpoint, ToolCall,
  Approval, Handoff, Trace, Artifact

Operations
  Deployment, Environment, Team / Namespace, Budget, Eval,
  Secret, ABOM
```

Identity resources change rarely and govern *what* an agent is. Runtime resources are created per execution and govern *what happened*. Operations resources govern *where, when, by whom, at what cost, and to what standard*.

## Conventions

Every resource follows the same envelope:

```yaml
apiVersion: joch.dev/v1alpha1
kind: <Kind>
metadata:
  name: string
  namespace: string
  labels: {}
  annotations: {}
spec: {}
status:
  phase: Pending | Ready | Running | Failed | Suspended
  conditions: []
  observedGeneration: number
```

The `status` subresource is updated by controllers, never by operators. Operators update `spec` and Joch reconciles.

## Where the catalog lives

Concrete YAML contracts are in [`kubernetes/`](kubernetes/index.md). The directory name reflects the **YAML style** (Kubernetes-conventions: apiVersion / kind / metadata / spec / status), not a runtime requirement — Joch resources work the same way against SQLite, Postgres, and Kubernetes CRDs.
