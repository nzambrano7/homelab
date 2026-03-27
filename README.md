# Homelab

Personal homelab running on Proxmox VE with Home Assistant for smart home automation.

## Stack
- **Hypervisor:** Proxmox VE 9.x on Lenovo ThinkCentre M920q
- **Smart Home:** Home Assistant OS (VM)
- **Network:** UniFi Cloud Gateway Ultra + USW Lite 8 PoE + U6+ APs
- **Zigbee:** SONOFF Zigbee 3.0 USB Dongle Plus via ZHA

## Structure
- `home-assistant/` - HA configuration and automations
- `network/` - Network design and VLAN documentation
- `proxmox/` - Proxmox setup and VM configs
- `docs/` - General documentation

## Goals
- Fully local smart home with zero cloud dependency
- Self-hosted services via Proxmox LXC containers
- Infrastructure as code — all config version controlled
