# Prefer Docker as the Default Minikube Driver


## Context

The project already standardizes on Minikube as the default local Kubernetes cluster. Minikube supports multiple drivers, including Docker, QEMU, VirtualBox, HyperKit, and others depending on the host environment.

Without a preferred driver, local setup may vary between developers. That increases the chance of inconsistent behavior, environment-specific troubleshooting, and extra onboarding friction. A default driver should align with the most common developer setup and require as little additional host-specific configuration as possible.

Docker is widely used in local development workflows and is commonly available on machines that already build or run containers. Using Docker as the default Minikube driver keeps the local setup closer to standard container tooling and avoids introducing an additional virtualization dependency in the common case.

## Decision

Docker should be the preferred default Minikube driver for local development.

When configuring Minikube, the default driver should be set with:

`minikube config set driver docker`

This makes Docker the standard baseline for project setup instructions unless a user has an environment-specific reason to choose a different driver.

## Consequences

Positive consequences:

- Local setup becomes more consistent across contributors using Minikube.
- The project can document a single default driver instead of leaving the choice unspecified.
- Many developers can use existing Docker installations instead of configuring an additional VM-based driver.

Negative consequences:

- Some environments may not have Docker installed or available.
- Certain host platforms or local constraints may still require a different Minikube driver.
- Troubleshooting guidance may still need to mention exceptions for non-Docker setups.

## Alternatives Considered

- **No preferred driver**: leaving the driver unspecified gives users maximum flexibility, but it reduces standardization and makes support guidance less consistent.

- **VM-based drivers such as VirtualBox or QEMU**: these can be appropriate in some environments, but they typically require extra host setup and are less aligned with common container-based local workflows.
