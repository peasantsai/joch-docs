# Joch Control Plane

Joch should be a domain-specific control plane that can use Kubernetes as one backend.

The architecture should look like this:

```text
joch CLI
  |
joch API / controller layer
  |
Kubernetes CRDs + controllers
  |
Pods, Jobs, Deployments, Secrets, Services
```

Joch resources can become Kubernetes custom resources:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-agent
spec:
  modelRef:
    name: gpt-5-thinking
  personalityRef:
    name: pragmatic-researcher
  tools:
    - web.search
    - github.create_issue
```

The controller reconciles that domain resource into lower-level Kubernetes objects:

```text
AgentDeployment
  |
Kubernetes Deployment
Kubernetes Secret
Kubernetes ServiceAccount
Kubernetes NetworkPolicy
Kubernetes ConfigMap
```

## Workload Mapping

Joch should compile agent concepts into infrastructure concepts:

```text
Execution
  |
Kubernetes Job
```

```text
Deployment
  |
Kubernetes Deployment / StatefulSet
```

```text
Schedule
  |
Kubernetes CronJob
```

This preserves Kubernetes scheduling, scaling, rollout, and health machinery while keeping the user-facing model agent-native.

## Product Responsibilities

Kubernetes answers:

```text
How do I run this workload?
Where should it be scheduled?
Is the container alive?
How many replicas are ready?
Can this pod access this secret?
```

Joch answers:

```text
What is this agent?
What model does it use?
What tools can it call?
What memory can it read?
What policy governs it?
What plan is it executing?
What did it do?
What did it cost?
Can it switch providers?
Can I replay or audit it?
```

That is the product layer.
