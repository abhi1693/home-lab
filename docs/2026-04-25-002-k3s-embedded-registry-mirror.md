# K3s Embedded Registry Mirror

## Status
Accepted

## Context
The cluster runs all workloads on three schedulable K3s server nodes. Large
images should not have to be pulled independently from upstream registries on
each node when another node already has them.

K3s includes an embedded distributed OCI registry mirror. It is enabled with
`embedded-registry: true` on server nodes, and each participating node must also
have mirror entries in `/etc/rancher/k3s/registries.yaml`.

## Decision
Enable the K3s embedded registry mirror through the `k3s_server` Ansible role.

Use a wildcard mirror entry:

```yaml
mirrors:
  "*":
```

Keep default upstream registry fallback enabled. This cluster is not air-gapped,
so falling back to the real registry is safer than failing image pulls when an
image is not yet available from another node.

## Operational Notes
All current Kubernetes nodes are K3s servers, so the `k3s_server` role writes the
required configuration everywhere today. Future agent nodes must receive the same
`registries.yaml` mirror configuration before they can participate.

Nodes must be able to reach each other on TCP ports `5001` and `6443` using their
internal addresses.

The embedded mirror shares images already present in node containerd stores. It
does not accept direct pushes.

## Security Notes
The embedded mirror assumes peer nodes are equally trusted. A node that can place
an image in its containerd store can advertise that tag to peers. Use image
digests instead of mutable tags when image integrity matters.
