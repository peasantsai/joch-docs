# CLI Reference

The `joch` CLI is the primary operator surface. Every command works against the resource store, the data plane, and the runtime adapter pinned by `joch context use ...`.

## Global flags

| Flag | Description |
|---|---|
| `--context <name>` | Override the active context for one command. |
| `--namespace, -n <ns>` | Scope command to a namespace. |
| `--output, -o <fmt>` | Output format: `table` (default), `wide`, `json`, `yaml`. |
| `--no-color` | Disable terminal color. |
| `--quiet, -q` | Suppress informational output. |
| `--debug` | Verbose debug logging. |
| `--server <url>` | Override `joch-server` URL. |

## Lifecycle

| Command | Description |
|---|---|
| `joch up` | Start a local control plane (SQLite, single process). |
| `joch down` | Stop a local control plane. |
| `joch doctor` | Self-check: API server, model creds, MCP reachability, worker health, trace endpoint. |
| `joch version` | Print client + server versions. |
| `joch init <kind>` | Generate a starter project (`local`, `docker-compose`, `kubernetes`). |
| `joch install <target>` | Install Joch into a target (`kubernetes` via Helm). |

## Contexts

| Command | Description |
|---|---|
| `joch context ls` | List contexts. |
| `joch context use <name>` | Select a context. |
| `joch context set <name> --server <url> [--namespace <ns>] [--auth <kind>]` | Define / update a context. |
| `joch context current` | Print the active context. |

## Resources

| Command | Description |
|---|---|
| `joch apply -f <file|dir>` | Create or update resources from YAML. |
| `joch validate -f <file|dir>` | Schema + selector + reference validation, no apply. |
| `joch diff -f <file|dir>` | Structural diff against current state. |
| `joch get <kind>` | List resources of a kind. Supports filters and selectors. |
| `joch describe <kind>/<name>` | Detailed view (spec + status + recent events). |
| `joch delete <kind>/<name>` | Delete a resource. |
| `joch label <kind>/<name> <key>=<value>` | Add or update labels. |
| `joch annotate <kind>/<name> <key>=<value>` | Add or update annotations. |
| `joch promote <kind>/<name> --from <env> --to <env>` | Run release gates and promote across environments. |
| `joch rollback <kind>/<name> --to-version <n>` | Re-apply a prior version. |

## Discovery

| Command | Description |
|---|---|
| `joch discover --framework <sdk> --path <path>` | Discover agents written in a vendor SDK and produce stub records. |

## Executions and runs

| Command | Description |
|---|---|
| `joch run <agent> <input>` | Trigger a one-shot execution. |
| `joch run <agent> -f <input.yaml>` | Trigger an execution with structured input. |
| `joch executions ls` | List executions. |
| `joch executions get <id>` | Inspect a single execution. |
| `joch executions cancel <id>` | Cancel a running execution. |
| `joch executions replay <id>` | Replay an execution (policy-gated). |
| `joch trace <id|last>` | Stream the trace for an execution. |

## Tool gateway

| Command | Description |
|---|---|
| `joch get toolcalls` | List recent ToolCalls (filters: `--agent`, `--tool`, `--since`, `--phase`). |
| `joch describe toolcall <id>` | Inspect a tool call (request, decision, result). |
| `joch toolcalls top --by <error_rate|cost|latency>` | Aggregate tool calls. |
| `joch toolcalls replay <id>` | Replay (policy-gated). |

## MCP gateway

| Command | Description |
|---|---|
| `joch mcp ls` | List registered MCP servers. |
| `joch mcp add <name> --endpoint <url> [--auth <kind>]` | Register a server. |
| `joch mcp discover <name>` | Refresh the discovered tool list. |
| `joch mcp pin <name> --to <version>` | Pin a server version. |
| `joch mcp upgrade <name> --to <version> --review` | Review the schema diff before upgrading. |
| `joch mcp quarantine <name> --reason <text>` | Take a server out of rotation fleet-wide. |
| `joch mcp policy set <name> --read-only` | Apply a server-level policy override. |
| `joch mcp trust <name>` | Mark a server as trusted (raises trust score). |

## Approvals

| Command | Description |
|---|---|
| `joch approvals ls` | List pending approvals. |
| `joch approvals decide <id> --decision <approve|reject|edit> [--rationale <text>]` | Decide an approval. |
| `joch approvals override <id> --decision approve --grant-by <user> --reason <text>` | Break-glass override. |

## Cost and observability

| Command | Description |
|---|---|
| `joch top agents` | Most active agents by execution count and cost. |
| `joch top tools` | Most-called tools, with success rate. |
| `joch top models` | Most-used models, by cost and latency. |
| `joch cost by-team [--since <window>]` | Cost rollup per team. |
| `joch cost by-agent [--since <window>]` | Cost rollup per agent. |
| `joch incidents ls` | Current incidents flagged by alerts. |
| `joch drift detect --agent <name>` | Detect output drift after a deployment. |
| `joch denials ls [--policy <name>] [--since <window>]` | Recent policy denials. |

## AgBOM

| Command | Description |
|---|---|
| `joch agbom <agent>` | Print the agent's AgBOM (default JSON). |
| `joch agbom <agent> --format <cyclonedx|spdx|swid>` | Emit a specific BOM format. |
| `joch agbom <agent> --diff --from <gen> --to <gen>` | Diff two AgBOM generations. |
| `joch agbom ls` | List agents and AgBOM generation. |
| `joch agbom ls --high-risk` | Filter agents whose AgBOM has high-risk components. |

## Memory and RAG

| Command | Description |
|---|---|
| `joch memories ls` | List memory bindings. |
| `joch describe memory <name>` | Inspect a memory record. |
| `joch rag ls` | List RAG indices. |
| `joch rag retrieve <name> --query <text>` | Run a retrieval interactively. |

## Eval and release

| Command | Description |
|---|---|
| `joch eval run <name> [--target <env>]` | Run an Eval against a target environment. |
| `joch eval ls` | List Evals and last results. |
| `joch promote <kind>/<name> --from <env> --to <env>` | Promote with release gates. |
| `joch rollback <kind>/<name> --to-version <n>` | Roll back to a prior version. |

## Policy

| Command | Description |
|---|---|
| `joch policy ls` | List policies. |
| `joch policy preview -f <file>` | Replay recent traffic against a draft policy. |

## Switching providers

| Command | Description |
|---|---|
| `joch agents switch <agent> --to <provider:model> [--conversation <id>] [--dry-run] [--apply]` | Migrate an agent (or a single conversation) across providers. |
| `joch route <execution-id> --explain` | Explain why the model router selected a particular candidate. |

## Output examples

```bash
joch get agents -o wide
joch get toolcalls --agent support-triage --since 1h -o json
joch describe agent support-triage -o yaml
```

## Shell completion

```bash
joch completion bash    >> ~/.bashrc
joch completion zsh     >> ~/.zshrc
joch completion fish    >> ~/.config/fish/completions/joch.fish
```
