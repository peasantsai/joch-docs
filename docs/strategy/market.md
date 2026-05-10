# Market Context

The AI agent ecosystem is consolidating around five forces:

```text
1. Vendor-native agent SDKs
2. Multi-agent orchestration frameworks
3. MCP-based tool integration
4. Enterprise agent governance platforms
5. Agent observability, evals, and deployment systems
```

## Competitive landscape

| Platform | Strength |
|---|---|
| OpenAI Agents SDK | Agent authoring, tools, handoffs, guardrails, structured outputs, tracing, OpenAI ecosystem. |
| Claude Agent SDK | Claude Code-style execution, tools, MCP, hooks, permissions, sessions. |
| Google ADK | Code-first agent development, debugging, deployment, evaluation, and multi-agent workflows. |
| Microsoft Agent Framework | Enterprise orchestration, MCP, A2A, observability, and Microsoft ecosystem integration. |
| LangGraph | Stateful graph orchestration for long-running agents. |
| LangSmith | Observability, tracing, evals, and monitoring. |
| CrewAI | Crews, tasks, flows, tools, memory, knowledge, and guardrails. |

## Where Joch AI can win

Joch AI wins by becoming the layer around these systems:

```text
agent inventory
MCP/tool-call gateway
policy enforcement
ABOM generation
tool-call audit
approval workflows
model routing metadata
cost accounting
agent release gates
framework/provider adapters
```

## Why MCP is the wedge

MCP is becoming shared connective tissue for tools and context. That creates a governance problem:

```text
unsafe local execution
prompt injection through tool results
supply-chain poisoning
insecure stdio handling
unknown tool sprawl
schema drift
unclear side effects
```

Joch AI turns that boundary into a controlled gateway.

## External references

- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/agents/)
- [OpenAI Agents SDK MCP guide](https://openai.github.io/openai-agents-python/mcp/)
- [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)
- [Google ADK](https://adk.dev/)
- [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/)
- [MCP security reporting](https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed)
