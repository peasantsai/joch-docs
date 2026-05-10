# Configuration Reference

Each Joch service reads configuration from CLI flags, environment variables, a config file, and (in Kubernetes) ConfigMap / Secret bindings. Precedence: CLI flag > env var > config file > Helm value > default.

## `joch-server`

```yaml
# /etc/joch/server.yaml
listen:
  address: 0.0.0.0
  port: 8080
  tls:
    certFile: /etc/joch/tls/tls.crt
    keyFile:  /etc/joch/tls/tls.key
auth:
  type: oidc
  oidcIssuer: https://accounts.example.com/realms/joch
  oidcAudience: joch-server
store:
  backend: postgres            # sqlite | postgres | kubernetes-crd
  postgres:
    dsn: postgres://joch:joch@postgres:5432/joch
    poolSize: 20
eventBus:
  backend: postgres            # postgres | redis | nats | kafka
audit:
  retentionDays: 365
  hashChain: true
admission:
  requireOwner: true
  defaultEnvironment: dev
controller:
  workers: 4
compiler:
  cacheSize: 256
abom:
  refreshOnChange: true
```

## `joch-worker`

```yaml
serverUrl: http://joch-server:8080
runtime:
  adapter: kubernetes          # local | docker | kubernetes | nomad | serverless
poolName: default
concurrency: 8
heartbeatSeconds: 15
shutdownGraceSeconds: 60
```

## `joch-gateway` (tool + MCP)

```yaml
serverUrl: http://joch-server:8080
listen:
  address: 0.0.0.0
  port: 8081
tool:
  rateLimit:
    burst: 50
    perSecond: 25
  redactionPatterns:
    - apikey:[A-Za-z0-9]{20,}
mcp:
  pinByDefault: true
  promptInjectionScan: true
  sandbox:
    cgroupCpuMillicores: 500
    cgroupMemoryMb: 256
    seccompProfile: /etc/joch/seccomp.json
```

## `joch-router` (model)

```yaml
serverUrl: http://joch-server:8080
healthCheck:
  intervalSeconds: 30
  syntheticPrompt: "Reply READY in JSON."
providers:
  openai:
    baseUrl: https://api.openai.com/v1
  anthropic:
    baseUrl: https://api.anthropic.com
  google:
    baseUrl: https://us-central1-aiplatform.googleapis.com
  microsoft:
    foundryEndpoint: https://your-foundry-service.services.ai.azure.com
  ollama:
    baseUrl: http://localhost:11434
streaming:
  bufferSize: 4096
```

## `joch-memory`

```yaml
serverUrl: http://joch-server:8080
backends:
  postgres:
    dsn: postgres://joch:joch@postgres:5432/joch
  pgvector:
    dsn: postgres://joch:joch@postgres:5432/joch
  redis:
    addr: redis:6379
  qdrant:
    url: http://qdrant:6333
default:
  workingTtlHours: 24
  semanticTopK: 5
```

## `joch-trace`

```yaml
listen:
  otlp:
    grpc: 4317
    http: 4318
backend:
  type: postgres
  postgres:
    dsn: postgres://joch:joch@postgres:5432/joch
  retentionDays: 30
export:
  ocsfSinks:
    - type: kafka
      bootstrapServers: kafka:9092
      topic: ocsf.events
sampling:
  default: tail
  rules:
    - match: "phase=failed"
      sample: 1.0
    - match: "joch.policy.deny=true"
      sample: 1.0
```

## `joch-operator` (Kubernetes)

Helm values:

```yaml
joch:
  mode: selfHosted
server:
  replicas: 2
  store:
    type: postgres
worker:
  replicas: 3
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 20
gateway:
  enabled: true
router:
  enabled: true
memory:
  enabled: true
  backend: pgvector
trace:
  otel: { enabled: true }
runtime:
  adapter: kubernetes
networkPolicies:
  enabled: true
serviceMonitor:
  enabled: true
```

## Per-resource defaults

Some defaults are applied at admission. They are configurable per [`Environment`](../specs/kubernetes/environment.md):

| Field | Default |
|---|---|
| `Trace.spec.sampling.mode` | `full` in `dev`/`staging`, `tail` in `prod` |
| `Trace.spec.retention.days` | 30 |
| `AgBOM.spec.refresh.onChange` | `true` |
| `AgBOM.spec.formats` | `[cyclonedx]` |
| `Budget.spec.enforcement.softLimitPct` | 0.8 |
| `Approval.spec.routing.timeoutMinutes` | 30 |
| `Policy.audit.logRequests` | `true` |
| `Policy.audit.redactSecrets` | `true` |
