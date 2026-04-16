# K3s Bootstrap Strategy

## Status
Accepted

## Context
The initial platform is:

- 3 Raspberry Pi 5 nodes
- K3s in HA mode
- Cilium instead of Flannel
- Ansible as the infrastructure control plane

The goal is to keep bootstrap simple, repeatable, and separable from day-2 cluster workloads.

## Decision
Bootstrap the platform in phases and keep each phase independently runnable.

## Bootstrap Phases

### 1. Host Preparation
Prepare the nodes before any K3s install work:

- base OS updates
- required packages
- Raspberry Pi tuning
- cgroup and kernel prerequisites
- time sync
- storage preparation

### 2. First Server Initialization
Create the cluster from a single initial server node.

This phase should establish:

- cluster token handling
- datastore mode
- core K3s configuration
- `tls-san` for the control-plane VIP

### 3. Control-Plane VIP
Bring up the control-plane VIP using `kube-vip`.

This phase creates the stable registration endpoint that all remaining nodes will use.

### 4. Additional Server Join
Join the remaining control-plane nodes to reach the 3-node HA layout.

All additional servers should join through `https://<vip>:6443`, not through the first server IP.

### 5. Agent Join
Add worker nodes later as a separate phase, not mixed into control-plane creation.

Agents should also join through `https://<vip>:6443`.

### 6. Kubeconfig Export
Make kubeconfig retrieval explicit and repeatable.

Exported kubeconfig should use the VIP as the API endpoint.

### 7. Cilium Installation
Install Cilium only after the base K3s control plane is reachable.

This should remain a separate step from K3s installation so networking changes stay isolated.

### 8. Cluster Validation
Run validation after bootstrap:

- node readiness
- etcd or control-plane health
- control-plane VIP reachability
- Cilium health
- DNS resolution
- service reachability checks

### 9. Platform Bootstrap
Only after the cluster is healthy should manifests from `kubernetes/` begin to apply.

## Design Rules

### Keep K3s Lean
Do not pack day-2 services into K3s bootstrap.

K3s bootstrap should focus on:

- cluster creation
- control-plane registration endpoint
- cluster networking handoff
- validation

### Keep Install, Upgrade, and Reset Separate
Use distinct operational entrypoints for:

- first install
- upgrade
- reset or rebuild

That keeps risky operations explicit.

### Prefer Explicit Re-Runs
Each phase should be safe to re-run without guessing which hidden state already exists.

## Assumptions

- Embedded etcd is used for HA
- Control-plane nodes run on reliable SSD-backed storage
- `kube-vip` provides the control-plane VIP
- Cilium replaces the default Flannel-based networking path

## Consequences

### Benefits

- Easier troubleshooting
- Cleaner upgrades
- No permanent dependency on the first control-plane node for future joins
- Clear boundary between cluster creation and cluster workloads

### Trade-Offs

- More playbooks or entrypoints than a single all-in-one bootstrap
- Slightly more operational discipline required
