# Pi-hole → AdGuard Home Migration — Handoff

Paste this at the start of your Claude Code session so it has full context before touching anything.

## Environment

- Proxmox host: `192.168.99.10` (Lenovo ThinkCentre M920q)
- UniFi network, VLANs: 1=Default, 10=Trusted, 20=IoT, 99=Management
- Repo: `github.com/nzambrano7/homelab` — commit as you go, CHANGELOG newest-first
- SSH access to Proxmox should already be set up from this machine

## Current state

| CT  | Service                              | IP            | VLAN |
|-----|---------------------------------------|---------------|------|
| 101 | Pi-hole + Unbound (recursive resolver) | 192.168.10.5 | 10   |
| 102 | docker-lxc (Debian 13) — Uptime Kuma (3001), Beszel hub (8090), Watchtower | 192.168.10.6 | 10 |

- Pi-hole was deployed via the community-scripts Proxmox helper, Unbound configured as recursive (not forwarding/DoT).
- Pi-hole has auto-update cron (`pihole -up`, Sundays 3am) and unattended-upgrades configured. `Restart=on-failure` (5s delay) already set.
- CT 102's `docker-compose.yml` lives at `/opt/monitoring/docker-compose.yml`.
- **Known gotcha:** new LXCs inherit Tailscale MagicDNS (100.100.100.100) from the Proxmox host by default — must manually override each container's DNS in the Proxmox DNS tab to `192.168.10.5` (or the new AdGuard IP once cut over).
- Network gateway swap (UCG Ultra → Cloud Gateway Fiber, WAS-110 XGS-PON module to fully bypass the AT&T BGW320) was in progress as of the last session. **Confirm this is fully complete and stable before starting the DNS migration** — the plan was deliberately to not run both changes at once.

## Decision already made: AdGuard Home over Technitium

Considered Technitium (full authoritative + recursive DNS, no Unbound needed) but chose AdGuard Home — bigger community (~27k vs ~5k GitHub stars), simpler for pure ad-blocking, and `adguardhome-sync` is purpose-built for two-instance redundancy. Technitium is worth revisiting later if split-horizon DNS or zone hosting ever becomes a need (e.g. new house, more complex network).

## Target architecture

| CT  | Service                              | IP            | VLAN |
|-----|---------------------------------------|---------------|------|
| 101 | Pi-hole + Unbound (decommission after cutover validated) | 192.168.10.5 | 10 |
| 103 | AdGuard Home Primary + Unbound (own instance, no shared dependency) | 192.168.10.7 | 10 |
| 104 | AdGuard Home Secondary + Unbound | 192.168.10.8 | 10 |
| 102 | + `adguardhome-sync` added to existing docker-compose stack | 192.168.10.6 | 10 |

`adguardhome-sync` mirrors blocklists, settings, and rewrites from primary → secondary. Only the primary (CT 103) gets managed directly.

## Migration order

1. Deploy CT 103 — AdGuard Home Primary + its own Unbound (recursive, same pattern as CT 101's)
2. Deploy CT 104 — AdGuard Home Secondary + its own Unbound
3. Add `adguardhome-sync` to CT 102's `docker-compose.yml`, point it at .7 → .8
4. Fix the Tailscale MagicDNS gotcha on both new CTs (Proxmox DNS tab override)
5. Update UniFi VLAN DHCP DNS settings: Primary → `192.168.10.7`, Secondary → `192.168.10.8`
6. Validate for a full day (DNS resolution, ad blocking, failover if primary drops)
7. Decommission CT 101 (Pi-hole)
8. Commit everything to the homelab repo, update CHANGELOG

## Working preferences

- Walk through CLI prompts step by step during installs rather than dumping a whole script
- Give honest complexity/trade-off assessments before committing to a path
- Casual, direct tone — no over-explaining fundamentals
