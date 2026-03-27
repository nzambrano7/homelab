# Network

UniFi network configuration documentation.

## Stack
- UniFi Cloud Gateway Ultra (router/firewall)
- USW Lite 8 PoE (managed switch)
- 2x U6+ Access Points

## VLANs
| VLAN | ID | Subnet | Purpose |
|------|-----|--------|---------|
| Trusted | 10 | 192.168.10.0/24 | Main devices |
| IoT | 20 | 192.168.20.0/24 | Smart devices |
| Management | 99 | 192.168.99.0/24 | Homelab gear |
