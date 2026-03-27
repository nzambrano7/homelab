# Changelog

All notable changes to this homelab will be documented here.

## [2026-03-26]
### Added
- Proxmox VE 9.x installed on Lenovo ThinkCentre M920q
- Home Assistant OS VM (ID 100) configured on Proxmox
- SONOFF Zigbee 3.0 USB Dongle passthrough to HA VM
- ZHA (Zigbee Home Automation) integration configured
- Studio Code Server add-on installed
- UniFi VLAN architecture (Trusted/IoT/Management)
- GitHub repo initialized with full folder structure

## [2026-03-27]
### Added
- Studio Code Server add-on installed with sidebar access
- GitHub repo initialized at github.com/nzambrano7/homelab
- Full repo structure with home-assistant, network, proxmox, docs folders
- CHANGELOG and README documentation added

### Decided
- Proxmox staying on Trusted VLAN 10 temporarily due to unmanaged switch limitation
- Will migrate to Management VLAN 99 when moving to new house with proper cable runs
