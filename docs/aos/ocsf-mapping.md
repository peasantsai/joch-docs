# OCSF Mapping

Joch emits security-relevant trace events as [Open Cybersecurity Schema Framework](https://ocsf.io/) records. SIEMs that already consume OCSF can ingest Joch events without bespoke parsers.

## Why OCSF

Agent fleets produce security-relevant signals: policy denials, approvals, MCP scanning hits, A2A messages, AgBOM updates, prompt-injection scan results. OCSF is the right schema for those signals because it is the schema security tools speak.

## Event class mapping

| Joch event | OCSF event class | UID |
|---|---|---|
| `PolicyDenied` | Application Activity | 8001 |
| `HookDecision` (deny / modify) | Application Activity | 8001 |
| `ApprovalRequested` | Application Activity | 8001 |
| `ApprovalGranted` / `ApprovalDenied` | Application Activity | 8001 |
| `ToolCallRequested` (external_write or higher) | Application Activity | 8001 |
| `ToolCallCompleted` (external_write or higher) | Application Activity | 8001 |
| `MCPSchemaDrift` | Application Activity | 8001 |
| `MCPQuarantined` | Application Activity | 8001 |
| `A2AMessageSent` / `A2AMessageReceived` | Network Activity | 4001 |
| `MemoryWritten` (PII / customer-tier) | Application Activity | 8001 |
| `KnowledgeRetrieved` (PII / regulated source) | Application Activity | 8001 |
| `AgBOMUpdated` | Process Activity | 1003 |
| `BudgetExceeded` | Application Activity | 8001 |
| `ProviderSwitched` | Application Activity | 8001 |

Operators can override the default event-class mapping per environment.

## Common attributes

Every Joch OCSF record carries:

```text
class_uid                    OCSF class id
type_uid                     concrete activity type id
time                         event time (UTC, milliseconds)
status                       success | failure | unknown
severity_id                  1 (Informational) ... 6 (Critical)
actor.user                   operator identity (when applicable)
actor.process.name           "joch-<service>"
actor.process.uid            joch service id
device                       runtime host / cluster / region
metadata                     Joch-specific tags (see below)
```

`metadata` includes `joch.agent`, `joch.execution`, `joch.tenant`, `joch.environment`, `joch.policy.id`, `joch.policy.version`, `joch.framework`.

## Example: PolicyDenied as OCSF Application Activity

```json
{
  "class_uid": 8001,
  "category_uid": 8,
  "category_name": "Application Activity",
  "class_name": "Application Activity",
  "type_uid": 800103,
  "type_name": "Application Activity: Other",
  "time": 1747893303123,
  "status": "Success",
  "status_id": 1,
  "severity_id": 3,
  "severity": "Medium",
  "actor": {
    "process": {
      "name": "joch-policy-engine",
      "uid": "policy-engine-1"
    }
  },
  "device": {
    "name": "joch-prod-eu",
    "type": "Server"
  },
  "metadata": {
    "version": "1.2.0",
    "product": { "name": "joch", "vendor_name": "PeasantsAI" },
    "tags": {
      "joch.agent": "support-triage",
      "joch.execution": "exec-20260510-001",
      "joch.tenant": "support-platform",
      "joch.environment": "prod",
      "joch.policy.id": "external-send-requires-approval",
      "joch.policy.version": "v3",
      "joch.hook.method": "steps/toolCallRequest",
      "joch.hook.decision": "deny",
      "joch.tool.name": "zendesk.create_ticket"
    }
  },
  "message": "Policy external-send-requires-approval@v3 denied steps/toolCallRequest for zendesk.create_ticket: approval required and not granted."
}
```

## Example: A2AMessageSent as OCSF Network Activity

```json
{
  "class_uid": 4001,
  "category_uid": 4,
  "category_name": "Network Activity",
  "class_name": "Network Activity",
  "type_uid": 400106,
  "type_name": "Network Activity: Other",
  "time": 1747893303456,
  "src_endpoint": { "agent": "support-triage" },
  "dst_endpoint": { "agent": "support-escalation" },
  "metadata": {
    "tags": {
      "joch.execution": "exec-20260510-001",
      "joch.handoff.id": "handoff-001",
      "joch.tenant": "support-platform"
    }
  },
  "message": "A2A handoff from support-triage to support-escalation."
}
```

## Sink configuration

```yaml
apiVersion: joch.dev/v1alpha1
kind: Trace
spec:
  export:
    ocsf:
      enabled: true
      eventClasses:
        - 8001
        - 1003
        - 4001
      sinkSecretRef:
        name: ocsf-sink
      transport: kafka
      topic: ocsf.events
```

Supported transports: HTTP, Kafka, AWS Kinesis, Splunk HEC, Elastic, Azure Event Hubs.

## Compatibility

Any SIEM or log aggregator that ingests OCSF accepts Joch records. Vendor-neutral schemas mean security teams do not need a Joch-specific parser to correlate agent activity with the rest of their telemetry.
