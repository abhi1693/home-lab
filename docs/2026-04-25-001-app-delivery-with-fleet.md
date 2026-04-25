# App Delivery With Rancher Fleet

## Status
Accepted

## Context
The cluster already uses Ansible to bootstrap the operating system, K3s,
Rancher, Cilium, Longhorn, and platform controllers.

Application manifests belong under `kubernetes/`, not inside Ansible roles.
The first application is Home Assistant under `kubernetes/apps/home-assistant/`.

## Decision
Use Rancher Fleet for application delivery.

Ansible remains responsible for bootstrapping the Fleet `GitRepo` object that
points Fleet at this repository. Fleet then reconciles app desired state from:

`kubernetes/apps/*`

## Operating Model
App install and upgrade flow:

1. Change app manifests under `kubernetes/apps/<app>/`.
2. Commit and push to the configured branch.
3. Fleet polls Git and reconciles the app bundle.
4. Validate the app rollout in Kubernetes or Rancher.

## Boundaries
Ansible owns:

- Fleet controller prerequisites through Rancher
- the Fleet `GitRepo` bootstrap resource

Fleet owns:

- app HelmChart resources
- app namespaces, services, ingress, PVCs, and policies

## Consequences
Benefits:

- App changes are Git-driven instead of copied manually to the server.
- Upgrades are reviewable as normal Git diffs.
- Rollbacks are Git reverts.

Trade-offs:

- Local uncommitted changes are not deployed by Fleet.
- The repo branch must be pushed before Fleet can reconcile new app state.
- Private repos require a Fleet Git credential secret.
