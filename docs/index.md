# Joch Docs

Joch is an agent control-plane concept: it owns agent identity, lifecycle, memory, policy, tools, artifacts, runtime state, and provider portability while treating OpenAI, Claude, Gemini, local models, and other vendors as inference backends.

## Start Here

<div class="grid cards" markdown>

-   :material-swap-horizontal: **State Portability**

    ---

    Define the vendor-neutral state model for switching providers mid-conversation without losing tools, memory, artifacts, or runtime state.

    [Read the strategy](architecture/state-portability.md)

-   :material-shape-outline: **Resource Model**

    ---

    Describe Kubernetes-style resource kinds for agents, models, memory, tools, execution, policy, guardrails, budgets, traces, and artifacts.

    [Explore the specs](specs/index.md)

-   :material-kubernetes: **Platform Strategy**

    ---

    Explain where Kubernetes should sit underneath Joch and why raw Kubernetes should not become the product abstraction.

    [Review the platform model](platform/index.md)

-   :material-server-network: **Service Architecture**

    ---

    Lay out control-plane and data-plane services, deployment modes, APIs, packaging, and runtime interfaces.

    [Open the architecture](architecture/service-architecture.md)

-   :material-tag-text-outline: **Brand**

    ---

    Capture the product naming direction and the language that should stay consistent across the Joch ecosystem.

    [Read the brand notes](product/brand.md)

-   :material-source-repository-multiple: **Repository Strategy**

    ---

    Map the proposed PeasantsAI GitHub repository system and the recommended order for splitting repos.

    [See the repo plan](product/repositories.md)

</div>

## Core Idea

!!! quote
    Vendors provide inference backends. Joch owns agent identity, memory, lifecycle, runtime state, policy, and auditability.

## Reading Path

1. Start with **State Portability** to understand the core architectural principle.
2. Continue to **Resource Model** for the CRD-style API surface.
3. Read **Kubernetes Strategy** before the deeper service architecture.
4. Finish with **Brand** and **Repository Strategy** when planning the public ecosystem.
