# Verify Local Docker Quota


## Context

The project already recommends explicit Minikube resource defaults for CPU, memory, and disk. Those values only work in practice if the underlying Docker environment is allowed to use at least that much capacity.

When Minikube uses Docker as its driver, the Minikube cluster runs inside the local Docker environment. If Docker is configured with lower limits than the Minikube defaults, the cluster may fail to start correctly or behave as if the requested Minikube resources were never actually available.

In this context, verifying the local Docker quota means checking that the Docker runtime available on the machine can provide at least the same CPU, memory, and disk capacity as the Minikube baseline. This should be treated as a prerequisite for reliable local setup.

This matters especially on WSL2-based setups, where Docker Desktop and the WSL2 VM may each introduce their own limits. A developer can configure Minikube with reasonable values and still get poor results if the underlying Docker environment remains more constrained.

## Decision

The project should recommend verifying that local Docker limits are not lower than the Minikube default resource baseline.

If the Minikube baseline is:

- 4 CPU
- 8 GB RAM
- 50 GB disk

then the local Docker environment should be able to provide at least those same values.

On WSL2-based setups, project guidance should include command examples to inspect the effective limits from inside the Linux environment, for example:

```bash
nproc
free -h
df -h /
```

These commands help verify:

- `nproc`: available CPU count
- `free -h`: available memory
- `df -h /`: available filesystem capacity

If the reported values are lower than the Minikube baseline, the Docker or WSL2 configuration should be adjusted before relying on the local cluster setup.

## Consequences

Positive consequences:

- Developers are more likely to notice host-level resource limits before debugging Minikube failures.
- Local cluster behavior becomes more predictable because Docker capacity is checked against the expected baseline.
- Troubleshooting guidance becomes clearer by distinguishing Minikube settings from Docker or WSL2 constraints.

Negative consequences:

- Setup instructions become slightly more detailed.
- Some users may need extra host-specific changes in Docker Desktop or WSL2 configuration before continuing.
- Reported filesystem capacity in WSL2 may require interpretation depending on how the local environment is provisioned.

## Alternatives Considered

- **Assume Docker can satisfy Minikube defaults**: this keeps setup shorter, but it hides an important source of local failures and inconsistent behavior.

- **Document only Minikube limits**: this partially standardizes the cluster configuration, but it ignores the lower-level runtime constraints that can invalidate those settings.

- **Provide guidance without command examples**: this is shorter, but it is less actionable, especially for users running Docker through WSL2.
