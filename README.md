# Cam's Homelab

My personal homelab environment for experminentation with infrastructure automation, GitOps workflows, Kubernetes, networking, and cloud-native tooling.

The goal of this lab is to provide a reproducible learning environment for testing and improving infra practices using modern DevOps tools and patterns.

## Overview

This homelab in the long term will house experiments and configurations for:

- Kubernetes orchestration with **k3s**
- Infrastructure Automation with **Ansible**
- Kubernetes package management with **Helm**
- GitOps deployments with **ArgoCD**
- Certificate management with TLS automation
- Networking, ingress, and service discovery.
- Monitoring and observability
- Configuration management and automation workflows
- **GitHub Actions** CI

## Repository Goals

- Keep infrastrucutre changes version controlled
- Automate repeatable idempotent tasks
- Practice GitOps principles
- Test new technologies safely
- Document infrastructure decisions and lessons learned

## Tech Stack

| Component | Purpose |
| --------- | ---------------------------------- |
| k3s | Lightweight Kubernetes Cluster |
| Ansible | Host provisioning and automation |
| Helm | Kubernetes application management |
| ArgoCD | GitOps continuous delivery |
| Git | Source of truth for infrastructure |
| Cert-manager | Certificate automation |
| Ingress Controller | External service routing |

## Deployment Model

This repository follows a GitOps workflow:

1. Infrastructure is defined as code
2. Changes are committed to Git
3. ArgoCD monitors the repository
4. Kubernetes automatically reconciles desired state

```
Git Repo
    |
    v
ArgoCD
    |
    v
K3s cluster
    |
    v
Running Services
```

## Long Term Philosophy and Goals

- Treat infrastrcure like cattle, not pets
- Never modify live systems manually
- Test, validate, and promote
- Git is the source of truth
- Automate everyothing repeatable
- Design for failure

