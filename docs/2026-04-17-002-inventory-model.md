# Inventory Model

## Status
Accepted

## Context
This repository uses two top-level boundaries:

- `infrastructure/`
- `kubernetes/`

The inventory model belongs under `infrastructure/` because it describes real machines and how automation targets them.

## Decision
Use one inventory per environment, starting with a single environment named `home`.

Recommended location:

`infrastructure/ansible/inventories/home/`

Recommended files:

- `hosts.yml`
- `group_vars/all.yml`
- `group_vars/k3s_servers.yml`
- `group_vars/k3s_agents.yml`
- `host_vars/<hostname>.yml`

## Modeling Rules

### Model Reality First
Inventory should describe physical and operational facts, not arbitrary app groupings.

Good group examples:

- `k3s_servers`
- `k3s_agents`
- `raspberry_pi`

Avoid groups like:

- `monitoring_nodes`
- `storage_things`
- `misc`

### Keep `host_vars` Small
Only store values that are truly host-specific:

- static IP
- MAC address if needed
- disk device name
- NIC name if it differs across nodes
- node labels or taints that apply only to one host

Everything shared by a role should live in `group_vars`.

### Keep `group_vars/all.yml` Stable
Put only global defaults there:

- timezone
- SSH user
- common package defaults
- cluster name
- common DNS or NTP settings

Do not turn `all.yml` into a dumping ground for every cluster option.

## Initial Group Shape

- `k3s_servers`: the three Pi5 control-plane nodes
- `k3s_agents`: worker nodes added later
- `raspberry_pi`: shared Raspberry Pi host preparation settings

## Rule Of Thumb

- If a value is true for one host, use `host_vars`.
- If a value is true for a role group, use that group’s `group_vars`.
- If a value is true for the whole environment, use `group_vars/all.yml`.

## Consequences

### Benefits

- Easy to reason about
- Easy to add more nodes later
- Keeps role logic separate from hardware facts

### Trade-Offs

- Some values may need to be moved as the lab grows
- Additional environments should get their own inventory instead of branching this one excessively
