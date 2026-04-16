# Homelab

Production-grade home infrastructure built on Proxmox VE, running virtualized 
and containerized workloads with enterprise network segmentation, DNS filtering, 
and remote access — all version controlled and documented.

## Infrastructure

| Layer | Technology |
|-------|-----------|
| Hypervisor | Proxmox VE 9.x on Lenovo ThinkCentre M920q |
| Smart Home | Home Assistant OS (QEMU VM) |
| DNS / Ad Blocking | Pi-hole + Unbound (LXC) — recursive DNS, no upstream dependency |
| Remote Access | Tailscale subnet router — full mesh VPN across all VLANs |
| Gateway | UniFi Cloud Gateway Ultra (UCG Ultra) |
| Switching | USW Lite 8 PoE x2 (laundry + office) |
| Wireless | U6+ APs x2 — dual-band, VLAN-aware |
| Zigbee | SONOFF Zigbee 3.0 USB Dongle Plus via ZHA |

## Network Architecture

Four-VLAN segmented network with Zone-Based Firewall and explicit allow/deny policies:

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 1 | Default | 192.168.0.0/24 | Infrastructure (switches, APs) |
| 10 | Trusted | 192.168.10.0/24 | Personal devices, servers |
| 20 | IoT | 192.168.20.0/24 | Smart home devices (isolated) |
| 99 | Management | 192.168.99.0/24 | Proxmox hypervisor |

**Security posture:** Block All inter-VLAN by default. IoT devices cannot reach 
Trusted or Management networks. Explicit allow rules for specific cross-VLAN 
services (DNS, Home Assistant API, Emulated Hue).

## Services

- **Pi-hole + Unbound** — Network-wide DNS filtering with recursive resolution. 
  No reliance on Cloudflare or Google DNS. Filters tracking and ad domains for 
  all Trusted devices.
- **Home Assistant** — Local smart home automation with Zigbee, TP-Link Kasa, 
  Alexa Media Player, and Emulated Hue integrations. Zero cloud dependency for 
  core automations.
- **Tailscale** — Subnet router exposing all four VLANs remotely. Full access 
  to Proxmox, HA, and Pi-hole from anywhere without port forwarding.
- **Emulated Hue** — HA acts as a Philips Hue bridge, enabling Alexa voice 
  control of local smart home entities across VLAN boundaries via mDNS proxy.

## Engineering Decisions

- **Proxmox on Management VLAN 99** — Hypervisor isolated from general traffic. 
  Access restricted to Trusted VLAN via explicit firewall rules (ports 8006, 22).
- **Pi-hole LXC on Trusted VLAN** — DNS server co-located with trusted devices. 
  IoT VLAN uses gateway DNS to avoid cross-VLAN firewall complexity.
- **Unbound recursive resolver** — Eliminates upstream DNS dependency. Queries 
  go directly to root nameservers, improving privacy and reducing attack surface.
- **Zone-Based Firewall** — Migrated from legacy Traffic Rules to UniFi 
  Zone-Based Firewall for cleaner policy enforcement and explicit zone isolation.
- **Tailscale over self-hosted WireGuard** — AT&T BGW320 double NAT prevents 
  inbound port forwarding. Tailscale traverses NAT without port exposure. 
  WAS-110 bypass planned for next house.
- **Git version control** — All configuration changes tracked in this repo with 
  dated changelog entries. Infrastructure treated as code.

## Planned Improvements

- WAS-110 XGS-PON SFP bypass + UCG Ultra Max at next house (eliminates AT&T gateway double NAT)
- Full rack build with UDM Pro/SE, USW Pro 24 PoE, Reolink NVR + Frigate
- Migrate remaining devices off Default VLAN 1
- Expand Proxmox with additional LXC workloads

## Repository Structure

- `home-assistant/` — HA configuration, automations, and integrations
- `network/` — VLAN design, firewall policy documentation, network diagrams
- `proxmox/` — Hypervisor configuration, VM/LXC specs
- `docs/` — Architecture decisions and runbooks
- `CHANGELOG.md` — Dated log of all infrastructure changes