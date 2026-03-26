# Remove Unused Profiles to Avoid Secrets Exposure


## Context

We already treat secrets exposure as a primary Minikube security risk. That risk does not end when a workload stops running.

Kubernetes stores Secret objects in etcd as part of cluster state. When Minikube uses the Docker driver, the cluster itself runs in Docker-managed containers and storage. In practice, that means secrets created in the cluster can remain present in the Docker-backed storage used by the Minikube profile until that profile is explicitly deleted.

An unused Minikube profile is therefore not just harmless leftover configuration. It can retain cluster state, including Secret data, certificates, and other sensitive Kubernetes objects, even when the profile is no longer actively used for development.

This creates an avoidable persistence risk:

- old local clusters may continue to hold credentials that are no longer needed
- developers may forget which profiles contain sensitive test or real integration credentials
- stale Docker-backed cluster storage may survive long after the original task or demo is finished
- local machines can accumulate unnecessary secret-bearing state across multiple Minikube profiles


## Decision

Unused Minikube profiles should be removed with `minikube delete` to reduce lingering secret exposure.

Project guidance should treat profile cleanup as a normal security practice, especially when Minikube uses the Docker driver.

Guidance should emphasize the following rules:

- delete Minikube profiles that are no longer needed instead of leaving them stopped but present
- prefer `minikube delete -p <profile>` for targeted cleanup of unused profiles
- use `minikube delete --all` when all local Minikube profiles are no longer needed
- treat profile deletion as the expected way to remove Docker-backed etcd state associated with old clusters
- avoid assuming that stopping a profile is enough to remove sensitive cluster data


## Consequences

Positive consequences:

- secret-bearing cluster state is less likely to remain on developer machines longer than necessary
- local Minikube usage better aligns with the principle of removing sensitive data when it is no longer needed
- developers are less likely to accumulate stale profiles with forgotten credentials and certificates
- cleanup guidance becomes concrete and operational instead of relying on users to remember hidden Docker-backed state

Negative consequences:

- developers may need to recreate profiles that they previously kept for convenience
- local setup can take longer when a deleted profile must be started again from scratch
- users may need clearer guidance on when to stop a profile temporarily versus when to delete it completely


## Alternatives Considered

- **Keep unused profiles and rely on developers to remember what they contain**: this is convenient, but it allows unnecessary secret-bearing cluster state to persist locally.

- **Stop profiles instead of deleting them**: this reduces active runtime usage, but it does not reliably remove the Docker-backed cluster storage that may still contain etcd data and secrets.

- **Treat profile cleanup as optional housekeeping**: this keeps instructions shorter, but it is too weak for a project that already recognizes secrets exposure as a primary local security risk.
