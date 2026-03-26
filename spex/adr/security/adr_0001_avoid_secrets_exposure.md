# Avoid Secrets Exposure


## Context

Minikube is commonly used for local Kubernetes development, testing, demos, and learning. Because it runs locally, users often treat it as a low-risk environment and apply weaker handling standards than they would in shared or production clusters.

That assumption is unsafe. A local Minikube cluster can still expose sensitive data through Kubernetes manifests, Helm values, shell history, mounted files, CI logs, terminal output, local image layers, and developer workstations themselves. The problem is often not a flaw in Minikube itself, but the way local development workflows normalize casual secret handling.

Common examples include:

- committing plaintext secrets into Git-tracked YAML files
- storing real credentials in `values.yaml` or `.env` files used for local deployments
- exposing secrets through `kubectl describe`, logs, or debugging output
- mounting host directories that contain cloud credentials, SSH keys, or kubeconfigs
- reusing production or shared-environment credentials in local clusters
- copying generated Secret manifests into tickets, chat, or documentation

These practices create a high-probability, high-impact failure mode. Once secrets are copied into source control, local files, shell history, or logs, they can persist long after the local cluster is deleted.


## Decision

Secrets exposure should be treated as one of the primary security risks when using Minikube.

General Minikube guidance should emphasize the following rules:

- prefer fake, disposable, or low-value credentials for local development
- do not commit plaintext secrets into manifests, Helm values, or other tracked files
- avoid reusing production credentials in Minikube
- minimize terminal, log, and debug output that reveals secret values
- avoid mounting sensitive host directories into pods unless explicitly required
- treat local kubeconfig files, cloud credentials, registry credentials, and SSH material as in-scope sensitive data when working with Minikube

When local secrets are necessary, guidance should prefer short-lived and low-privilege credentials over long-lived shared ones.


## Consequences

Positive consequences:

- local Kubernetes guidance better reflects a real security threat instead of assuming local means safe
- developers are less likely to leak credentials through Git, logs, screenshots, and shell history
- Minikube workflows become safer to use on laptops, shared workstations, and demo environments
- teams are more likely to build good secret-handling habits that also transfer to higher environments

Negative consequences:

- local setup may become slightly less convenient when fake or short-lived credentials must be created
- some tutorials and ad hoc examples become longer because they can no longer inline real secret values
- users may need additional tooling or process guidance for managing local development secrets safely


## Alternatives Considered

- **Treat secret handling as a secondary concern in Minikube**: this keeps local guidance simpler, but it ignores one of the most common ways local Kubernetes environments leak sensitive data.

- **Focus only on cluster hardening topics such as RBAC or pod privileges**: those topics matter, but they do not address the frequent day-to-day exposure path caused by careless handling of credentials in local workflows.

- **Assume local development credentials are low impact**: this may be true in some cases, but it is too weak as a general rule because local environments often end up using real cloud, registry, or third-party service credentials.
