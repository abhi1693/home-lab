# Control-Plane Registration Endpoint

## Status
Accepted

## Context
This lab will start with 3 Raspberry Pi 5 control-plane nodes running K3s in HA mode with embedded etcd.

Additional servers and agents should not join through a specific control-plane node such as `server-1`, because that creates an unnecessary special case in bootstrap, kubeconfig access, and future operations.

## Decision
Use `kube-vip` to provide a stable control-plane VIP for the K3s API and registration endpoint.

Rules:

- bootstrap the first server directly
- bring up the kube-vip control-plane VIP immediately after the first server is available
- join additional servers through `https://<vip>:6443`
- join agents through `https://<vip>:6443`
- point exported kubeconfig at the VIP
- include the VIP in `tls-san` on all K3s server nodes

## Scope
`kube-vip` is used only for the control-plane registration and API VIP.

It is not the default service `LoadBalancer` implementation for this repo.

Service exposure remains a separate concern and is handled by Cilium design decisions under `kubernetes/`.

## Why This Option

- avoids making the first server a permanent special case
- keeps the registration endpoint stable during server maintenance
- works without introducing two additional dedicated load-balancer hosts
- fits a small Pi-based HA cluster

## Alternatives Considered

### Join Through The First Server
Rejected because it couples future joins and kubeconfig access to one node identity.

### HAProxy And Keepalived
Valid, but not selected for the initial design.

It would add extra infrastructure and is a better fit if dedicated external load-balancer nodes are available and desired.

## Consequences

### Benefits

- cleaner bootstrap flow
- consistent kubeconfig endpoint
- simpler future expansion

### Trade-Offs

- kube-vip becomes part of the control-plane bootstrap story
- VIP and certificate handling must be correct from the beginning
