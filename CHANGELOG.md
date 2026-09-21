# Changelog

All notable changes to this homelab will be documented here.
## [2026-09-21]

### Fixed
- UniFi Cloud Gateway Fiber's "Encrypted DNS" feature was silently overriding all configured upstream DNS (both the VLAN 10 DHCP setting and WAN manual DNS), forwarding every query to a local DoH proxy on `127.0.0.1:5053` regardless of what was configured elsewhere. Disabled Encrypted DNS and set WAN DNS Server to Manual (`192.168.10.7` primary / `192.168.10.8` secondary). Confirmed at the dnsmasq config level (`/run/resolv.conf.d/main`) and via direct client testing that ad-blocking now works end-to-end (client → gateway → AdGuard → Unbound). This likely explains why Pi-hole's ad-blocking may have silently stopped working after the UCG Ultra → Cloud Gateway Fiber swap too — nobody tested blocking specifically after that change.
- Confirmed no secondary/failover WAN uplink exists that could silently regress this fix (`eth4` interface is a dormant leftover from the pre-bypass AT&T BGW320 handoff, not an active failover).

### Pending
- Full-day validation of the AdGuard Home cutover, restarted from today since ad-blocking wasn't actually flowing through AdGuard until this fix
- Decommission CT 101 (Pi-hole) once validation passes
- `proxmox/README.md`'s VM/CT table is stale — only lists VM 100, missing CT 101–104

## [2026-09-19]

### Added
- `adguardhome-sync` added to CT 102's docker-compose stack (`ghcr.io/bakito/adguardhome-sync`) — mirrors blocklists, settings, and rewrites from CT 103 (origin) to CT 104 (replica) every 10 minutes via cron. Credentials in a local `.env` file, not committed.
- UniFi VLAN 10 DHCP DNS Server updated to `192.168.10.7` (primary) / `192.168.10.8` (secondary), replacing Pi-hole. Fixed IP reservations added in UniFi for CT 103/104's MAC addresses since both addresses fall inside the DHCP pool range (`192.168.10.6`–`.254`).

## [2026-09-18]

### Added
- CT 103 "adguard-primary" created — Debian 13, `192.168.10.7/24` (VLAN 10), 1 vCPU, 512MB RAM, 2GB disk, unprivileged, nesting+keyctl enabled, protected. AdGuard Home deployed via community-scripts, paired with its own Unbound instance (recursive, same config as CT 101's) at `127.0.0.1:5335`. Fallback DNS (`1.1.1.1`, `9.9.9.9`) configured for resilience Pi-hole never had. Private reverse DNS pointed at the UniFi gateway (`192.168.10.1`) instead of Pi-hole.
- CT 104 "adguard-secondary" created — same spec as CT 103, `192.168.10.8/24`. Own independent Unbound instance, no shared dependency on CT 103.
- Both CTs' own OS-level DNS resolution pointed at themselves (`127.0.0.1`) instead of Pi-hole, for full self-sufficiency.

### Decided
- AdGuard Home chosen over Technitium for the Pi-hole replacement — bigger community, simpler for pure ad-blocking, and `adguardhome-sync` is purpose-built for primary/secondary redundancy. Technitium worth revisiting later if split-horizon DNS or zone hosting becomes a need.

## [2026-06-22]

### Fixed
- Uptime Kuma Proxmox monitor — switched from HTTPS to TCP Port 8006; pveproxy worker recycling was causing false positive alerts every 30–60 min. TCP check is unaffected by worker cycling.
- Beszel Proxmox host status alert delay increased to 2 min — brief WebSocket reconnects during pveproxy recycling were triggering false alerts.
- e1000e NIC watchdog hang returned after Proxmox kernel update (7.0.12-1-pve). BIOS update alone was insufficient. Permanent fix applied: `echo "options e1000e SmartPowerDownEnable=0" > /etc/modprobe.d/e1000e.conf` + `update-initramfs -u -k all`. Applied to all installed kernels.

### Added
- Watchtower container added to monitoring stack in CT 102 — auto-updates Uptime Kuma and Beszel Hub daily at 4am. Requires DOCKER_API_VERSION=1.40 for Docker Engine 29.x compatibility.
- Pi-hole weekly auto-update cron (`pihole -up` every Sunday at 3am) + unattended-upgrades for Debian security patches.
- Home Assistant auto-update enabled via Settings → System → Updates.

### Updated
- Proxmox VE, Pi-hole, and Home Assistant OS all updated.

## [2026-06-21]

### Added
- CT 102 "docker-lxc" created — Debian 13, 192.168.10.6/24 (VLAN 10), 2 vCPU, 1024MB RAM, 8GB disk, unprivileged, nesting enabled. DNS manually overridden to 192.168.10.5 — Tailscale MagicDNS was taking over by default.
- Docker + Docker Compose installed in CT 102.
- Uptime Kuma deployed (port 3001) — monitors Pi-hole, Home Assistant, Proxmox, Internet/DNS. Discord webhook notifications on all monitors.
- Beszel Hub deployed (port 8090) — agent on Proxmox host (binary, port 45876, auto-updates enabled). Alerts: Status 2min delay, CPU/Memory >90%, Disk >85%, Temp >80°C, Load Avg >4 — all 10min delay. Discord via Shoutrrr format.

### Fixed
- Proxmox NIC watchdog crashes (e1000e) — partially resolved by BIOS update to M1UKT79A/1.0.0.121 via USB flash. Note: use exFAT not FAT32 for USB drives >32GB in Windows diskpart.

### Decided
- AT&T 5G fiber upgrade declined — UCG Ultra caps at 1G WAN, full multi-gig requires ~$1000 in new hardware for ~1 year remaining stay.
- WAS-110 XGS-PON module + UniFi Cloud Gateway Fiber ($279) ordered for current house — eliminates BGW320-505 double NAT now. BGW320-505 masquerade values pre-extracted and documented. Hardware in transit.

### Pending
- WAS-110 + Cloud Gateway Fiber install
- Samsung TV / Tesla VLAN migration to IoT VLAN 20
- Unbound prefetch fix for Brave/Discord DNS hang (192.168.10.63)

## [2026-04-16]
### Changed
- Migrated to Zone-Based Firewall — IoT VLAN moved to Untrusted zone for proper isolation
- mDNS Proxy changed from Auto to Custom (IoT + Trusted) — required for Emulated Hue/Alexa discovery across VLANs
- CyberSecure Encrypted DNS disabled — was bypassing Pi-hole entirely
- Default Security Posture changed to Block All
- IGMP Snooping enabled on all VLANs
- Pi-hole set as DNS for Trusted, Default, and Management VLANs
- IoT DNS left on Auto (gateway) — Pi-hole filtering not needed for IoT devices

### Added
- Allow IoT DNS to Pi-hole — Untrusted → Internal, TCP/UDP port 53
- Allow IoT to HA Emulated Hue — Untrusted → Internal 192.168.10.204, ports 80/1900
- Allow Internal to IoT — Internal → Untrusted, all traffic
- Block Default to All — Internal/Default → Trusted + IoT + Management

### Removed
- All legacy Simple firewall rules replaced by Zone-Based Firewall policies
- Block IoT to Main — replaced by Untrusted zone isolation
- Allow Trusted to IoT — replaced by Allow Internal to IoT
- HA to IoT — redundant with Allow Internal to IoT
- Allow Management to IoT — redundant with Allow Internal to IoT

### Fixed
- Alexa discovery of Emulated Hue devices — fixed by Custom mDNS Proxy scope
- IoT device DNS resolution — Zone-Based Firewall allows IoT → Pi-hole on port 53
- Bambu Lab LAN connection from PC — fixed by Allow Internal to IoT rule

### Known Issues
- UniFi APs on Default VLAN blocked from Pi-hole — not critical, APs don't need DNS filtering
- Samsung TV still on Trusted VLAN — move to IoT when convenient
- Tesla on Trusted VLAN (offline) — move to IoT when it reconnects

## [2026-04-14]
### Added
- Tailscale installed on Proxmox (pve) for remote access
- Subnet routes advertised: 192.168.1.0/24, 192.168.10.0/24, 192.168.20.0/24, 192.168.99.0/24
- iPhone added as Tailscale peer
- Remote access confirmed working for Proxmox (100.104.61.48:8006) and HA (192.168.10.204:8123)

### Removed
- WireGuard LXC (CT 102) removed — redundant with Tailscale
- UCG Ultra port forward UDP 51820 removed

### Decided
- AT&T BGW320 double NAT prevents true IP passthrough to UCG Ultra
- Tailscale handles remote access short-term
- WAS-110 + UCG Ultra Max planned for new house to eliminate AT&T gateway entirely

## [2026-04-13]
### Added
- Automatic HA backups configured — daily schedule, 3 copies retained, encrypted with key stored in 1Password Homelab vault
- Pi-hole LXC (ID 101) deployed on Proxmox via community-scripts
  - Static IP: 192.168.10.5/24 on Trusted VLAN 10
  - Unbound installed as recursive DNS resolver (no upstream dependency)
  - 87,771 domains blocked on default lists
- All VLANs (Default/1, Trusted/10, IoT/20) DHCP DNS updated to 192.168.10.5
- WireGuard LXC (ID 102) deployed on Proxmox via community-scripts
  - Static IP: 192.168.10.6/24 on Trusted VLAN 10
  - WGDashboard v4.3.3 installed for peer management
  - wg0 tunnel configured on 10.0.0.1/24, listen port 51820
  - Phone peer added (10.0.0.2/32)
  - Port forward created on UCG Ultra — UDP 51820 → 192.168.10.6

### Blocked
- WireGuard remote access non-functional due to AT&T gateway double NAT
  - UCG Ultra WAN IP is 192.168.1.244 (private), not public IP
  - Short-term fix: Tailscale (pending)
  - Long-term fix: WAS-110 SFP bypass at new house

## [2026-04-02]
### Added
- Garage lights schedule automation (on at 19:00, off at 07:00) via time trigger
- Exposed light.garage_bulb_1 and light.garage_bulb_2 to Emulated Hue in configuration.yaml

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

## [2026-03-27]
### Added
- Studio Code Server add-on installed with sidebar access
- GitHub repo initialized at github.com/nzambrano7/homelab
- Full repo structure with home-assistant, network, proxmox, docs folders
- CHANGELOG and README documentation added

### Decided
- Proxmox staying on Trusted VLAN 10 temporarily due to unmanaged switch limitation
- Will migrate to Management VLAN 99 when moving to new house with proper cable runs

## [2026-03-26]
### Added
- Proxmox VE 9.x installed on Lenovo ThinkCentre M920q
- Home Assistant OS VM (ID 100) configured on Proxmox
- SONOFF Zigbee 3.0 USB Dongle passthrough to HA VM
- ZHA (Zigbee Home Automation) integration configured
- Studio Code Server add-on installed
- UniFi VLAN architecture (Trusted/IoT/Management)
- GitHub repo initialized with full folder structure