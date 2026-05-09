# Brand

# **joch**
Meaning:
A Joch is the wooden frame used to bind and guide oxen or draft animals so they can pull together.

Product metaphor:

Tagline:
> **joch: Joch harnesses AI agents into coordinated working fleets.**

That maps cleanly to our product:

```text
joch agents
joch models
joch skills
joch memories
joch executions
joch deployments
```

It is also more brandable than `joch`, which sounds like a utility command rather than a platform.

CLI:

```bash
joch get agents
joch run researcher "Analyze this repo"
joch apply -f agent.yaml
joch trace exec-123
joch approvals ls
```

Kubernetes-style alias, optional:

```bash
jochctl get agents
```

But I would make the primary command:

```bash
joch
```

Cleaner, memorable, less derivative.

---

# Brand architecture

```text
Company / org:     Peasants AI
Product:           joch
CLI:               joch
Cloud:             joch Cloud
Runtime:           joch Runtime
Kubernetes:        joch Operator
Specs:             joch Specs
Registry:          joch Registry
```

Domain-style naming if we later need it:

```text
joch.dev
joch.run
joch.ai
```

API group:

```yaml
apiVersion: joch.peasants.ai/v1alpha1
kind: Agent
```

or cleaner:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
```

---

# Other good names

If we want alternatives before committing:

| Name        | CLI       | Feel                                                   |
| ----------- | --------- | ------------------------------------------------------ |
| **joch**  | `joch`  | Fleet assembly, strong, memorable                      |
| **Yoke**    | `yoke`    | Control/coordination, short, unusual                   |
| **Flock**   | `flock`   | Multi-agent swarm/fleet, friendly                      |
| **Field**   | `field`   | Peasants AI theme, broad platform feel                 |
| **Acre**    | `acre`    | Infrastructure/territory metaphor                      |
| **Tiller**  | `tiller`  | Cultivation/control, but Helm had old “Tiller” baggage |
| **Herd**    | `herd`    | Agent fleet, direct but less polished                  |
| **Orchard** | `orchard` | Grows/cultivates agents, softer                        |
| **Mason**   | `mason`   | Builds agent systems                                   |
| **Forge**   | `forge`   | Builds/deploys agents, common but strong               |

My pick remains:

```text
joch
```

It is short, command-like, and directly communicates “manage the fleet.”

---
