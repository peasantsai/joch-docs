# Platform

Joch should use Kubernetes as an infrastructure backend, not as the product abstraction exposed to every user.

The platform model is:

```text
Use Kubernetes as the infrastructure substrate.
Build Joch as the agent control plane above it.
```

Not:

```text
Rebuild Kubernetes from scratch.
```

And not:

```text
Force users to model agents directly as Pods, Deployments, ConfigMaps, and Jobs.
```

## Sections

<div class="grid cards" markdown>

-   :material-kubernetes: **Kubernetes Role**

    ---

    Explain why Kubernetes is useful underneath Joch, and why it is not enough as the agent product model.

    [Read the Kubernetes role](kubernetes.md)

-   :material-sitemap-outline: **Joch Control Plane**

    ---

    Define the domain-specific layer that understands agents, models, tools, memory, policy, and execution.

    [Review the control plane](control-plane.md)

-   :material-state-machine: **CRDs and Controllers**

    ---

    Show how Kubernetes custom resources and controllers can implement the production backend.

    [Open the CRD design](crds-controllers.md)

-   :material-source-branch: **Deployment Modes**

    ---

    Separate local developer mode from cluster production mode without changing the resource specs.

    [Compare deployment modes](deployment-modes.md)

-   :material-compass-outline: **Product Strategy**

    ---

    Keep the CLI backend-agnostic and let the same specs work locally, in clusters, and later in managed SaaS.

    [Read the strategy](product-strategy.md)

</div>

## Platform Rule

Users should describe agent intent. Joch should compile that intent into the right infrastructure objects for the selected backend.

```text
User-facing:
  Agent, Skill, Model, Memory, Execution, ToolCall, Policy

Control-plane:
  Joch controllers and compilers

Infrastructure-facing:
  Kubernetes Jobs, Deployments, Secrets, Services, ConfigMaps
```

The user should run:

```bash
joch run research-agent --task "Analyze this market"
```

They should not need to manually construct a Kubernetes Job with model, prompt, memory, and policy configuration wired together by hand.
