# Repo Boundaries

## Status
Accepted

## Scope
This repository currently has two top-level boundaries:

- `infrastructure/`
- `kubernetes/`

## Boundary Definitions

### `infrastructure/`
Owns everything required to create, prepare, and operate the underlying home-lab platform.

Examples:

- Ansible inventories, playbooks, and roles
- Raspberry Pi OS preparation
- K3s bootstrap and upgrade workflows
- Cilium installation and cluster-level bootstrap
- Node-level storage preparation
- Network and edge configuration that is not expressed as Kubernetes manifests
- Secrets handling for infrastructure automation

### `kubernetes/`
Owns desired state that should live inside the cluster.

Examples:

- Cluster bootstrap manifests after the base cluster exists
- Namespaces, platform services, and policies
- Ingress, Gateway API, and service exposure manifests
- Observability stack
- Application deployment manifests
- GitOps-managed resources if GitOps is introduced later

## Rule Of Thumb

- If it prepares machines or creates the cluster, it belongs in `infrastructure/`.
- If it is applied to the running cluster as Kubernetes resources, it belongs in `kubernetes/`.

## Documentation Convention

Documentation lives under `docs/`.

Filename format:

`YYYY-MM-DD-NNN-title.md`

Examples:

- `2026-04-17-001-repo-boundaries.md`
- `2026-04-17-002-inventory-model.md`
- `2026-04-17-003-cilium-service-exposure.md`

Rules:

- Use a real date in ISO format
- Use a three-digit sequence number for documents created on the same date
- Use lowercase kebab-case for the title
- Prefer one decision or topic per document
- Create a new document when a decision changes materially instead of rewriting history

## Immediate Next Structure

Start small and expand only when needed:

### `infrastructure/`

- `ansible/`
- `networking/`
- `secrets/`

### `kubernetes/`

- `bootstrap/`
- `platform/`
- `apps/`

## Notes

Networking remains under `infrastructure/` for now to preserve the two-boundary model.
