# Set Reasonable Resource Defaults for Minikube


## Context

Minikube uses default quotas for local cluster resources unless a user overrides them explicitly. In this context, default quotas are the baseline CPU, memory, and disk values assigned to the Minikube cluster for local use. They define how much capacity is available to Kubernetes system components and to workloads deployed into the cluster.

At the time of writing, Minikube defaults to 2 CPU and 20000 MB disk when those values are not overridden explicitly. Memory is left to Minikube's automatic default selection unless it is configured directly. These defaults are conservative and should be treated as the baseline this project wants to override.

If these defaults are too small, local environments become unstable or misleading. Developers may see pods stuck in `Pending`, repeated evictions, image pull and unpack failures caused by limited disk, or poor performance under even modest multi-service workloads. That creates unnecessary troubleshooting noise and makes the local setup less representative of practical project usage.

This matters especially for setups that include heavier infrastructure components such as local Kafka. Kafka and similar stateful or memory-hungry services can quickly exhaust conservative local defaults, especially when combined with supporting services, application pods, and Kubernetes overhead.

## Decision

The project should recommend resource defaults for Minikube closer to:

- 4 CPU
- 8 GB RAM
- 50 GB disk

These values can be configured with:

```bash
minikube config set cpus 4
minikube config set memory 8192
minikube config set disk-size 50000mb
```

This means overriding the smaller built-in baseline of 2 CPU and 20000 MB disk, and replacing Minikube's automatic memory default with an explicit 8 GB allocation.

These values should be treated as the standard baseline for local development guidance unless a specific workflow can justify smaller or larger limits.

The intent is not to maximize resource usage, but to choose defaults that are reasonable for modern local Kubernetes development. A baseline around 4 CPU, 8 GB RAM, and 50 GB disk provides enough headroom for Kubernetes itself plus common application stacks and infrastructure services, including local Kafka-based setups.

When configuring Minikube, the project guidance should prefer explicit resource settings rather than relying on minimal or tool-provided defaults.

## Consequences

Positive consequences:

- Local environments become more capable of running realistic multi-service workloads.
- Developers are less likely to hit avoidable resource-related failures during setup or testing.
- Project setup becomes more predictable because cluster capacity is no longer left to conservative defaults.
- Stateful and infrastructure-heavy components such as Kafka are more likely to run successfully in the default local environment.

Negative consequences:

- The recommended baseline requires a moderately capable developer machine.
- Some users may need to reduce the values on resource-constrained laptops or shared environments.
- A larger default allocation may consume more host resources than lightweight workflows strictly need.

## Alternatives Considered

- **Keep smaller defaults**: this reduces host resource usage, but it makes the default setup fragile for realistic workloads and increases the chance of misleading local failures.

- **Do not recommend explicit quotas**: leaving sizing unspecified gives users flexibility, but it weakens standardization and makes local setup outcomes less consistent.

- **Recommend much larger defaults**: larger allocations could support even heavier workloads, but they would exclude more developer machines without strong evidence that such a baseline is needed for common local use.
