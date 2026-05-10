# Platform

Joch's "platform" section explains where Kubernetes fits underneath Joch and how the deployment story works. The primary architectural treatment lives in [Architecture](../architecture/index.md); these pages focus on the operations side: how to install, where Kubernetes ends, and how local / Docker / cluster modes compose.

## Sections

<div class="grid cards" markdown>

-   :material-kubernetes: **Kubernetes Role**

    ---

    Why Kubernetes is the right substrate underneath Joch, and why raw Kubernetes is not sufficient as the agent product surface.

    [Read the Kubernetes role](kubernetes.md)

-   :material-state-machine: **CRDs and Controllers**

    ---

    How Joch resources can be implemented as Kubernetes CRDs with operators, and what controllers reconcile.

    [Read the CRD design](crds-controllers.md)

-   :material-source-branch: **Deployment Modes**

    ---

    Local developer mode, Docker Compose, Kubernetes, and managed SaaS — same resource specs, different runtime.

    [Compare deployment modes](deployment-modes.md)

-   :material-compass-outline: **Product Strategy**

    ---

    Backend-agnostic CLI, the same specs across environments, room for managed SaaS without forking the model.

    [Read the strategy](product-strategy.md)

</div>

## Platform rule

> Users describe agent intent. Joch compiles that intent into the right infrastructure objects for the selected backend.

```text
User-facing            Agent, FrameworkAdapter, Tool, MCPServer, Memory, Policy, Eval, ...
Control-plane          Joch controllers and compiler
Infrastructure-facing  Kubernetes Jobs, Deployments, Services, Secrets, ConfigMaps,
                       or Docker containers, or local processes
```

Operators run:

```bash
joch run support-triage --task "Triage the queue"
```

— not handwritten Kubernetes Jobs with model, prompt, memory, policy, and trace export wired together.
