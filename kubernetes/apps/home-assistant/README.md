# Home Assistant

Home Assistant is installed through the upstream `pajikos/home-assistant`
Helm chart. Rancher Fleet reconciles this app path from Git, and K3s'
Helm controller installs the chart from the `HelmChart` resource.

The initial endpoint is:

- `http://ha.home`

That hostname needs an internal DNS record or local hosts entry pointing to the
Traefik LoadBalancer IP, `192.168.3.3`.

Current choices:

- chart: `home-assistant`
- chart version: `0.3.56`
- app version: `2026.4.4`
- HACS version: `2.0.5`
- namespace: `home-assistant`
- ingress class: `traefik`
- persistence: Longhorn, `1Gi`
- code-server add-on: enabled as a sidecar at `http://code.ha.home`

HACS is bootstrapped by the `install-hacs` init container in
[helmchart.yaml](/home/asaharan/PycharmProjects/home-lab/kubernetes/apps/home-assistant/helmchart.yaml).
The init container only installs HACS when `/config/custom_components/hacs` is
missing, so HACS UI-managed updates are not overwritten on normal pod restarts.
If the Home Assistant PVC is kept, HACS survives reinstall and pod recreation. If
the PVC is deleted or rebuilt, the init container installs the pinned HACS
version again before Home Assistant starts.

Hardware mounts remain disabled until there is a concrete device workflow to
expose.

## Code Server

The chart's code-server add-on runs as a sidecar against the Home Assistant
`/config` volume and is exposed through Traefik at:

- `http://code.ha.home`

The chart default runs code-server with `--auth none`, so keep this hostname
limited to the trusted internal network.

## UniFi AP PoE schedule

Use the built-in UniFi Network integration for switch-port PoE control. The
automation is managed from Git as a Home Assistant package in
[packages-configmap.yaml](/home/asaharan/PycharmProjects/home-lab/kubernetes/apps/home-assistant/packages-configmap.yaml).

1. In UniFi OS, create a local-only admin with Network full management access.
   Cloud/SSO users do not work for Home Assistant's UniFi Network integration.
2. In Home Assistant, add `UniFi Network` from
   `Settings > Devices & services > Add integration`.
3. Open the `USL24PB` UniFi switch device in Home Assistant and enable the
   disabled PoE port control entity for port 13. Use the PoE port entity, not the
   power-cycle button.
4. Commit and push changes under `kubernetes/apps/home-assistant/`. Fleet applies
   the ConfigMap and the Home Assistant StatefulSet mounts it at
   `/config/packages`.

Home Assistant packages are enabled by `configuration.templateConfig`, and the
init container also enforces `homeassistant.packages: !include_dir_named
packages` in the persisted `/config/configuration.yaml`.

## Person tracking

The `Abhimanyu Saharan` person is managed from Git in
[packages-configmap.yaml](/home/asaharan/PycharmProjects/home-lab/kubernetes/apps/home-assistant/packages-configmap.yaml)
with these device trackers:

- `device_tracker.abhi_pc`
- `device_tracker.abhimanyu_pixel_8`

Home Assistant login users are stored in HA's auth storage, not in package YAML.
Create or update the login user `asaharan` from `Settings > People > Users`, then
link it to the `Abhimanyu Saharan` person in the People UI. If you want that user
link in Git later, copy the Home Assistant user `ID` from the Users tab and add
it as `user_id` on the `person` entry.

The package uses `switch.usw_24_poe_port_13_poe`; confirm the exact entity ID
from the port 13 entity settings in Home Assistant.

The `button.usw_24_poe_port_13_power_cycle` entity only restarts port 13
briefly. It cannot keep the AP powered off from midnight until 05:00.
The `sensor.usw_24_poe_port_13_poe_power` entity only reports current PoE power
draw. It confirms Home Assistant can see the port, but it cannot control power.

Time triggers do not run retroactively. If the automation is created after
midnight, the off action waits until the next midnight. To test immediately, run
`switch.turn_off` against `switch.usw_24_poe_port_13_poe` from Home Assistant's
Actions developer tool, then run `switch.turn_on` to restore power.

The StatefulSet is annotated for Stakater Reloader. If Reloader is running,
changes to `home-assistant-packages` trigger a Home Assistant pod restart so
package changes are loaded from Git without manually editing Home Assistant.

Do not schedule the only AP that provides access to Home Assistant or UniFi
unless both services remain reachable over wired networking while Wi-Fi is off.
