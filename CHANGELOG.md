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

## [2026-03-30]
### Added
- HACS (Home Assistant Community Store) installed
- Alexa Media Player integration (8 devices, 46 entities) — Echo Show 10, Echo Spot, Fire TV Sticks x3, Samsung QLED TV
- Emulated Hue integration configured for Alexa voice control of HA entities
- Kasa HS103 (Espresso Machine) connected via TP-Link Smart Home integration
- UniFi firewall rule: Allow IoT to HA on port 80 for Emulated Hue discovery
- HACS integrations documented in docs/hacs_integrations.md

### Decided
- Alexa as temporary voice platform — switching to Apple HomeKit + HomePod Minis in new house
- Smart bulbs over switches for current house given ~1 year timeline
- Second USW Lite 8 PoE ordered for desk — USW Lite 8 PoE stays in laundry for AP PoE
- custom_components excluded from git tracking — HACS reinstallable, not portfolio value

## [2026-04-01]
### Added
- USW Lite 8 PoE installed at desk (office switch)
- THIRDREALITY Zigbee RGBCW bulbs paired to ZHA (garage)

### Changed
- Proxmox migrated from Trusted VLAN 10 (192.168.10.10) to Management VLAN 99 (192.168.99.10)
- vmbr0 configured as VLAN-aware bridge with bridge-pvid 99
- vmbr0.99 created as Proxmox management interface
- HA VM network device set to vmbr0 with VLAN tag 10, maintaining 192.168.10.204
- Office switch Port 3 configured as trunk (Native VLAN 99, Tagged VLAN 10)
- Laundry switch uplink port set to Native VLAN Default (1) with Allow All — fixes VLAN tag corruption on return traffic

### Fixed
- HA internet outage caused by laundry switch uplink port having Native VLAN Trusted (10) instead of Default (1), corrupting gateway ARP reply VLAN tags

### Decided
- Uplink/trunk ports between switches must always be Native Default (1) — they are pass-through pipes, not endpoints