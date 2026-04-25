# Home Assistant

Home Assistant is installed through the upstream `pajikos/home-assistant`
Helm chart. Rancher Fleet reconciles this app path from Git, and K3s'
Helm controller installs the chart from the `HelmChart` resource.

The initial endpoint is:

- `http://home-assistant.home`

That hostname needs an internal DNS record or local hosts entry pointing to the
Traefik LoadBalancer IP, `192.168.3.3`.

Current choices:

- chart: `home-assistant`
- chart version: `0.3.56`
- app version: `2026.4.4`
- namespace: `home-assistant`
- ingress class: `traefik`
- persistence: Longhorn, `10Gi`

Home Assistant add-ons such as code-server and hardware mounts are intentionally
left disabled until there is a concrete device or editing workflow to expose.
