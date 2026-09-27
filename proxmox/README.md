# Proxmox

Proxmox VE 9.x running on Lenovo ThinkCentre M920q.

## Hardware
- CPU: Intel i5-8500T (6 cores)
- RAM: 16GB DDR4 2666 dual channel
- Storage: 1TB Samsung PM981 NVMe

## VMs
| ID | Name | Purpose |
|----|------|---------|
| 100 | homeassistant | Home Assistant OS |

## Containers (LXC)
| ID | Name | Purpose | IP | VLAN |
|----|------|---------|----|------|
| 102 | docker-lxc | Docker host — Uptime Kuma, Beszel Hub, Watchtower, adguardhome-sync | 192.168.10.6 | 10 |
| 103 | adguard-primary | AdGuard Home Primary + own Unbound instance | 192.168.10.7 | 10 |
| 104 | adguard-secondary | AdGuard Home Secondary + own Unbound instance | 192.168.10.8 | 10 |
| 105 | valheim | Valheim dedicated game server (Docker) | 192.168.10.9 | 10 |
