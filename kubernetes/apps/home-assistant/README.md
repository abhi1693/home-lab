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
- persistence: Longhorn, `1Gi`

Home Assistant add-ons such as code-server and hardware mounts are intentionally
left disabled until there is a concrete device or editing workflow to expose.

## UniFi AP PoE schedule

Use the built-in UniFi Network integration for switch-port PoE control.

1. In UniFi OS, create a local-only admin with Network full management access.
   Cloud/SSO users do not work for Home Assistant's UniFi Network integration.
2. In Home Assistant, add `UniFi Network` from
   `Settings > Devices & services > Add integration`.
3. Open the `USL24PB` UniFi switch device in Home Assistant and enable the
   disabled PoE port control entity for port 13. Use the PoE port entity, not the
   power-cycle button.
4. Paste
   [examples/unifi-ap-poe-schedule.yaml](/home/asaharan/PycharmProjects/home-lab/kubernetes/apps/home-assistant/examples/unifi-ap-poe-schedule.yaml)
   into a new automation. The example uses `switch.usw_24_poe_port_13_poe`;
   confirm the exact entity ID from the port 13 entity settings in Home
   Assistant.

The `button.usw_24_poe_port_13_power_cycle` entity only restarts port 13
briefly. It cannot keep the AP powered off from midnight until 05:00.
The `sensor.usw_24_poe_port_13_poe_power` entity only reports current PoE power
draw. It confirms Home Assistant can see the port, but it cannot control power.

Time triggers do not run retroactively. If the automation is created after
midnight, the off action waits until the next midnight. To test immediately, run
`switch.turn_off` against `switch.usw_24_poe_port_13_poe` from Home Assistant's
Actions developer tool, then run `switch.turn_on` to restore power.

Do not schedule the only AP that provides access to Home Assistant or UniFi
unless both services remain reachable over wired networking while Wi-Fi is off.
