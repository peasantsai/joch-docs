# CrewAI Integration

[**CrewAI**](https://docs.crewai.com/) is a multi-agent framework for orchestrating crews of role-based agents. Joch attaches to it through the `crewai` framework adapter.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `Agent` | Role-based agent: `role`, `goal`, `backstory`, `tools`, `llm`, `memory`. |
| `Task` | Unit of work: `description`, `expected_output`, `agent`, `tools`, `context`. |
| `Crew` | Collection of agents and tasks. |
| `Process` | `sequential`, `hierarchical` (with a manager LLM). |
| `Flow` | Pythonic workflow control inside a crew. |
| Tools | LangChain-compatible tools, custom tools, MCP integrations. |
| Memory | Short-term, long-term, entity, contextual. |
| Telemetry | Opt-in CrewAI telemetry; logs and OpenTelemetry compatibility. |

## Mapping table

| CrewAI | Joch |
|---|---|
| `Agent(role, goal, ...)` | One [`Agent`](../specs/kubernetes/agent.md) record per CrewAI agent, all sharing `framework.adapterRef: crewai` |
| `Task(...)` | One [`Execution`](../specs/kubernetes/execution.md) per task |
| `Crew(agents=[...], tasks=[...], process=...)` | A parent `Agent` with `framework.workflow.shape=crew` and child agent records |
| `Process.hierarchical` | The manager LLM is recorded as a separate `Agent`; subordinates report via `Handoff` |
| Tools | One [`Tool`](../specs/kubernetes/tool.md) record per tool |
| MCP tools | [`MCPServer`](../specs/kubernetes/mcpserver.md) record |
| Memory backends | Bridged to [`Memory`](../specs/kubernetes/memory.md) when delegated |
| LLM selection | Routed through the [Model Router](../architecture/model-router.md) |

## The smallest install

### Before

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(role="Researcher", goal="Find market evidence", llm="gpt-5-thinking", tools=[web_search])
writer    = Agent(role="Writer",      goal="Draft the report",   llm="gpt-5-thinking")

t1 = Task(description="Research agent ops market", agent=researcher, expected_output="bullets")
t2 = Task(description="Write the report",          agent=writer,     context=[t1])

crew = Crew(agents=[researcher, writer], tasks=[t1, t2], process=Process.sequential)
print(crew.kickoff())
```

### After — Joch-governed

```python
from crewai import Agent, Task, Crew, Process
from joch.adapters.crewai import bind_runtime

# ... same agents/tasks/crew as before ...

crew = bind_runtime(
    Crew(agents=[researcher, writer], tasks=[t1, t2], process=Process.sequential),
    joch_agent_ref="research-crew",
    server_url="http://joch-server:8080",
)
print(crew.kickoff())
```

`bind_runtime(...)` injects:

- a tool wrapper that emits AOS hooks and applies `Policy`,
- a model wrapper that routes through the Joch model router,
- a memory wrapper that writes to [`Memory`](../specs/kubernetes/memory.md) where delegated,
- a `Crew` event interceptor that emits one `Execution` per `Task`, plus `Handoff` events between agents,
- trace export to Joch trace with `joch.*` attributes.

## What Joch leaves to the SDK

- `role` / `goal` / `backstory` semantics.
- Sequential / hierarchical process orchestration.
- Native CrewAI memory implementations.
- Manager-LLM driven coordination in `Process.hierarchical`.

## Reference

- CrewAI docs: <https://docs.crewai.com/>
