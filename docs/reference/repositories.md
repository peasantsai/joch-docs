# Repositories

## Initial repos

```text
github.com/peasantsai/joch
github.com/peasantsai/joch-spec
github.com/peasantsai/joch-docs
github.com/peasantsai/joch-examples
github.com/peasantsai/joch-registry
```

## Later repos

```text
github.com/peasantsai/joch-operator
github.com/peasantsai/joch-mcp
github.com/peasantsai/joch-js
github.com/peasantsai/joch-python
github.com/peasantsai/joch-ui
github.com/peasantsai/joch-adapters
github.com/peasantsai/joch-cloud
```

## Main repo structure

```text
joch/
  cmd/
    joch/
    joch-server/
    joch-gateway/
    joch-worker/

  api/
    openapi/
    proto/
    schemas/

  pkg/
    apiserver/
    auth/
    store/
    registry/
    gateway/
    policy/
    approvals/
    abom/
    trace/
    discovery/
    adapters/
      openai/
      anthropic/
      langgraph/
      crewai/
      mcp/
    runtime/
      local/
      docker/
      kubernetes/

  deploy/
    docker-compose/
    helm/
    kubernetes/
    local/

  examples/
    basic/
    mcp-gateway/
    langgraph/
    crewai/
    openai-agents/
    claude-agent-sdk/

  docs/
  scripts/
  tests/
```
