# Joch Docs

Joch is an agent control-plane concept: it owns agent identity, lifecycle, memory, policy, tools, artifacts, runtime state, and provider portability while treating OpenAI, Claude, Gemini, local models, and other vendors as inference backends.

## Structure

- **State Portability** defines the vendor-neutral state model for switching providers mid-conversation.
- **Resource Model** describes Kubernetes-style resource kinds for agents, models, memory, tools, execution, policy, guardrails, budgets, traces, and artifacts.
- **Kubernetes Strategy** explains where Kubernetes should sit underneath Joch and why raw Kubernetes should not be the product abstraction.
- **Service Architecture** lays out control-plane and data-plane services, deployment modes, APIs, packaging, and runtime interfaces.
- **Brand** captures the product naming direction.
- **Repository Strategy** maps the proposed PeasantsAI GitHub repository system.
