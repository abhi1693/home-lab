# Cilium Service Exposure

## Status
Accepted

## Context
This lab will use Cilium as the cluster networking layer. Service exposure needs a clear default so ingress and LoadBalancer behavior do not become ad hoc over time.

The control-plane VIP is a separate concern and is handled by `kube-vip`, not by Cilium.

The repo currently has only two top-level boundaries:

- `infrastructure/`
- `kubernetes/`

That means:

- edge router and home-network configuration stay under `infrastructure/`
- Kubernetes-native exposure objects stay under `kubernetes/`

## Decision
Use this layered model:

### Layer 1: Edge And Network Reachability
Managed under `infrastructure/`.

Examples:

- router or Unifi-side routing
- BGP peering
- VLAN reachability
- DNS forwarding or upstream records

### Layer 2: Cluster LoadBalancer Behavior
Managed in Kubernetes through Cilium-native resources.

Examples:

- LoadBalancer IP pools
- L2 announcements for services
- BGP advertisement for services
- service advertisement policy

### Layer 3: Application Entry
Managed in Kubernetes through standard workload-facing resources.

Examples:

- Gateway API
- Ingress
- Services
- TLS and certificates

## Default Rule

- If the change talks to the home network or router, it belongs in `infrastructure/`.
- If the change advertises or allocates service IPs inside the cluster, it belongs in `kubernetes/`.
- If the change exposes an app, it belongs in `kubernetes/`.
- If the change is about the control-plane API VIP, it is not part of this document and follows the `kube-vip` decision.

## Recommended Starting Position

Use one default exposure path and avoid mixing patterns early.

Selected progression:

1. Use Cilium LB IPAM for service VIP allocation
2. Use Cilium BGP Control Plane to advertise selected `LoadBalancer` VIPs to the edge router
3. Use bundled K3s Traefik as the standard ingress controller for north-south HTTP entry

## Things To Avoid Early

- Mixing multiple ingress strategies at once
- Using both ad hoc NodePorts and LoadBalancers as the default exposure pattern
- Putting router configuration inside Kubernetes manifests
- Mixing control-plane VIP ownership with service exposure ownership
- Letting each app define a different exposure model without a platform standard

## Open Questions

- Will Gateway API replace `Ingress` as the default north-south entry layer later?
- Which DNS source of truth will own internal service names?

## Consequences

### Benefits

- Clear ownership boundary
- Clear separation between control-plane access and service exposure
- Easier troubleshooting of edge versus cluster issues
- Cleaner future migration to GitOps-managed platform networking
- Stable service reachability without tying ingress to a specific node IP

### Trade-Offs

- Some exposure work will always span both top-level boundaries
- Edge router BGP configuration must be maintained outside Kubernetes
- Service exposure now depends on both cluster and router-side correctness
