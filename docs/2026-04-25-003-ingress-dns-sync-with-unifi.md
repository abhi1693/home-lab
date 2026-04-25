# Ingress DNS Sync With UniFi

## Status
Accepted

## Context
Ingress hostnames should not require matching manual DNS entries in UniFi. The
Kubernetes Ingress object should be the source of truth for internal app names.

ExternalDNS supports watching Ingress resources and writing DNS records through a
webhook provider. UniFi is supported by the
`kashalls/external-dns-unifi-webhook` provider.

## Decision
Run ExternalDNS as a Fleet-managed app with the UniFi webhook provider.

ExternalDNS watches only `Ingress` resources for the `traefik` ingress class and
only reconciles hostnames under the `home` suffix. DNS records are written to
UniFi with `policy: sync` and TXT ownership records.

The UniFi API key is not stored in Git. It must exist as
`kube-public/external-dns-unifi-secret` before the Fleet app is pushed.

## Consequences
Adding an Ingress host in Git creates the matching UniFi DNS record. Removing an
Ingress host removes the DNS record if it is owned by this ExternalDNS instance.

Manual UniFi DNS records outside ExternalDNS ownership are left alone.

UniFi DNS is backed by dnsmasq, so wildcard and duplicate CNAME behavior is
limited by UniFi.
