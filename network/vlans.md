- **HA VM:** bridge `vmbr0`, VLAN tag `10` → gets IP on Trusted VLAN
- **Pi-hole LXC:** bridge `vmbr0`, VLAN tag `10` → static `192.168.10.5`

## Firewall Policy (Zone-Based)

### Zones

| Zone | Networks |
|------|----------|
| Internal | Default, Trusted, Management |
| Untrusted | IoT |
| External | WAN (Internet 1, Internet 2) |

### Zone Matrix (default behavior)

| Source → Destination | Policy |
|---------------------|--------|
| Internal → Internal | Allow All |
| Internal → Untrusted | Allow All |
| Internal → External | Allow All |
| Untrusted → Internal | Block All |
| Untrusted → External | Allow All |
| Untrusted → Gateway | Allow All |
| External → Internal | Allow Return only |
| External → Untrusted | Allow Return only |

### Custom Rules

| Rule | Source | Destination | Port | Protocol | Action |
|------|--------|-------------|------|----------|--------|
| Allow IoT DNS to Pi-hole | Untrusted | 192.168.10.5 | 53 | TCP/UDP | Allow |
| Allow IoT to HA Emulated Hue | Untrusted | 192.168.10.204 | 80, 1900 | TCP/UDP | Allow |
| Allow Internal to IoT | Internal | Untrusted | Any | All | Allow |
| Allow Trusted DNS to Pi-hole | Internal/Trusted | 192.168.10.5 | 53 | TCP/UDP | Allow |
| Allow Trusted to Proxmox | Internal/Trusted | 192.168.99.10 | 8006, 22 | TCP | Allow |
| Block Default to All | Internal/Default | Trusted + Management | Any | All | Block |

## Design Decisions

**Why separate Management VLAN for Proxmox?**
Hypervisor access is restricted to Trusted devices only via explicit firewall 
rules. A compromised IoT device cannot reach the Proxmox web UI or SSH.

**Why IoT in its own zone?**
Zone-Based Firewall provides cleaner policy enforcement than legacy Traffic Rules. 
IoT devices are completely isolated from Trusted and Management by default, with 
only specific exceptions (DNS, Emulated Hue) explicitly allowed.

**Why Pi-hole on Trusted VLAN instead of a dedicated DNS VLAN?**
Simplicity — Pi-hole serves Trusted devices which is the highest-value use case. 
IoT devices use gateway DNS to avoid cross-VLAN firewall complexity. A dedicated 
DNS VLAN or running Pi-hole on the gateway itself would be the next iteration.

**Why Tailscale instead of self-hosted WireGuard?**
AT&T BGW320 uses IP Passthrough (not true bridge mode), causing double NAT that 
prevents inbound port forwarding to the UCG Ultra. Tailscale traverses NAT 
without requiring open ports. Self-hosted WireGuard is planned once the AT&T 
gateway is bypassed with a WAS-110 SFP module at the next house.

**Uplink port rule: always Native VLAN 1**
Trunk/uplink ports between switches must always use Native VLAN 1 (Default). 
Setting a data VLAN as native on an uplink corrupts return traffic VLAN tags — 
learned the hard way when HA lost internet after a switch misconfiguration.