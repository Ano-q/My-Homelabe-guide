# Homelab

A single-node Proxmox homelab that runs storage, media, photos, monitoring, a game server and four independent remote-access paths for a household of about twenty users.

This repository documents what was built, why each design choice was made, and how to rebuild every piece yourself. Each project is self-contained and ends with a prompt you can paste into an AI assistant to rebuild it in your own environment.

> All addresses, names and identifiers in this repository are placeholders. The lab uses the private ranges listed in [docs/architecture.md](docs/architecture.md), the domain `example.com`, and documentation-range public IPs such as `203.0.113.10`.

## What is in the lab

- **One Proxmox VE host** (`pve-01`): dual Xeon, 96 GB ECC RAM, an RTX 4060 Ti passed through to the media VM, and a 10 GbE uplink.
- **Edge network**: the ISP modem in bridge mode behind an Omada ER8411 router and an Omada 10G switch.
- **Storage**: an OpenMediaVault VM that owns two software RAID arrays, and ZFS pools for backups.
- **Services in VMs and LXC containers**: DNS filtering, a TLS reverse proxy, WireGuard, Tailscale, Prometheus and Grafana, Immich, Jellyfin, and a modded Minecraft server.
- **Remote access** that does not depend on one vendor: WireGuard as the primary path, Tailscale as the fallback, an outbound-only relay for networks that block VPNs, and a home-hosted REALITY edge for a school network.

## Projects

Each project is a build someone else can reproduce. Start with the architecture overview, then pick any project.

| Project | What it shows |
|---|---|
| [Proxmox base host](projects/proxmox-base-host/) | Dual-socket host, NUMA, GPU passthrough, host timers |
| [NAS and storage](projects/nas-storage/) | OMV VM with passed-through disks, RAID10 and RAID5, private NFS bridge, Immich |
| [Backup strategy](projects/backup-strategy/) | Nightly vzdump to ZFS pools, tiered retention, discard and trim |
| [Network edge](projects/network-edge/) | Bridged ISP modem, Omada router and switch, 10G, renumbering with rollbacks |
| [DNS, reverse proxy and TLS](projects/dns-proxy-tls/) | Split-horizon DNS with AdGuard, Caddy wildcard certificate by DNS-01, DDNS |
| [Tiered WireGuard VPN](projects/wireguard-tiered-vpn/) | In-kernel WireGuard in an unprivileged LXC, four server-enforced access tiers |
| [Host hardening](projects/host-hardening/) | Lynis-driven hardening, nftables host firewall with timed auto-rollback |
| [Monitoring and alerting](projects/monitoring-alerting/) | Prometheus, Grafana, Glance, push alerts and an external dead-man switch |
| [UPS graceful shutdown](projects/ups-nut-shutdown/) | NUT on a consumer UPS, self-healing USB link watchdog |
| [VPN-routed media stack](projects/vpn-media-stack/) | Gluetun namespace killswitch, GPU transcoding, hardlink-safe layout |
| [Censorship-resistant relay](projects/censorship-resistant-relay/) | Outbound-only WireGuard to a VPS running Xray REALITY, narrow blast radius. Includes the [school network edge](projects/censorship-resistant-relay/school-network-edge.md): REALITY on TCP 443 at home |
| [Isolated game server](projects/game-server-isolation/) | Pelican Panel, container egress isolation, tunnel instead of port forward |

## Design principles

These show up in every project.

1. **Verify from outside.** A port forward is tested from an external network, a firewall change from a fresh connection, an alert by causing a real failure.
2. **Arm the undo before the change.** Risky remote changes run behind a timed automatic rollback.
3. **Enforce access on the server.** Client configs are routing hints. Access rules live where the user cannot edit them.
4. **Pin what you cannot afford to break.** Images that hold databases are pinned by version or digest.
5. **Write the decision rule down while the evidence is fresh.** Thresholds and verdicts are recorded before they are needed.

## Repository layout

```
.
├── README.md                 this file
├── docs/
│   └── architecture.md       topology, addressing, guests, VLAN plan
└── projects/
    └── <project>/README.md   one self-contained build per folder
```

## How to use the "Rebuild with AI" prompts

Every project README ends with a prompt. Copy it into an AI assistant, then answer its first questions with your own values: subnets, host IDs, domain. The prompts never contain real secrets, and they ask the assistant to generate keys on your machine rather than inventing them.
