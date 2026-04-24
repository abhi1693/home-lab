# UniFi BGP

This directory holds the edge-router side of the cluster service exposure design.

The cluster uses:

- Cilium LB IPAM for allocating the ingress VIP and app-service LoadBalancer VIPs
- Cilium BGP Control Plane for advertising that VIP
- bundled K3s Traefik as the ingress controller behind the advertised VIP

The UDM-side configuration is manual. UniFi expects an FRR-format BGP configuration file to be uploaded.

Use [udm-pro-bgp.conf](/home/asaharan/PycharmProjects/home-lab/infrastructure/network/unifi/udm-pro-bgp.conf).

Current repo-aligned assumptions:

- UDM local ASN: `65000`
- UDM LAN/gateway IP: `192.168.3.1`
- K3s/Cilium local ASN: `65001`
- Traefik LoadBalancer IP: `192.168.3.3`
- App LoadBalancer pool: `192.168.3.16-192.168.3.23`
- K3s peers:
  - `192.168.3.243` (`k8s-rpi1`)
  - `192.168.3.191` (`k8s-rpi2`)
  - `192.168.3.108` (`k8s-rpi3`)

Operational note:

- If the UDM firewall policy is restrictive for LAN-to-gateway traffic, allow TCP `179` from the three node IPs to the UDM.
