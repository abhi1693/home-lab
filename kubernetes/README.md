# Kubernetes Desired State

This directory owns resources that are applied to the running K3s cluster after
the infrastructure bootstrap has completed.

Rancher Fleet reconciles app manifests from this directory after the
`fleet_apps` Ansible role has bootstrapped the Fleet `GitRepo`.

For day-to-day app changes:

1. Edit files under `kubernetes/apps/<app>/`.
2. Commit and push to the configured Fleet branch.
3. Let Fleet reconcile the app bundle.

K3s `HelmChart` resources should live in `kube-system`; their
`targetNamespace` controls where the chart itself is installed.
