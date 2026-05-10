# Concepts

These pages establish the shared vocabulary and product position of Joch before the architecture, resources, and operator workflows. Read them in order if you are new to the project.

## Reading order

<div class="grid cards" markdown>

-   :material-numeric-1-circle-outline: **Positioning**

    ---

    What Joch is, what it deliberately is not, and why "another agent SDK" is the wrong category for it.

    [Read the positioning](positioning.md)

-   :material-numeric-2-circle-outline: **Five Pillars**

    ---

    The product is organized around inventory, governance, portability, observability, and release management.

    [Read the five pillars](five-pillars.md)

-   :material-numeric-3-circle-outline: **Comparison**

    ---

    How Joch sits relative to OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, and CrewAI.

    [Read the comparison](comparison.md)

-   :material-numeric-4-circle-outline: **Glossary**

    ---

    Definitions for every term used across the rest of the docs — agent record, framework adapter, AgBOM, AOS, tool gateway, model route, hook, and more.

    [Read the glossary](glossary.md)

</div>

## Core idea, in one paragraph

Vendor SDKs let you **build** an agent. Joch lets you **operate** every agent your organization runs — across SDKs, vendors, and environments. Joch owns the records, gateways, policies, traces, costs, deployments, and release gates that the SDKs leave to you. Once Joch holds the cross-SDK metadata graph, the policy enforcement point, the portable execution and state model, and the trace and eval history, replacing it becomes harder than replacing the SDK underneath. That is the moat.
