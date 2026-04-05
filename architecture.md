# Architecture

## Overview

The server is an old laptop running Proxmox VE as the hypervisor. All services run as LXC containers, keeping them isolated and easy to manage independently.

All services are **internal-only**; no ports are forwarded on the router. Remote access is handled entirely by Tailscale.

---

## How Access Works

### On the home network (LAN)
```
Device
        |
    Pi-hole (local DNS)
        |
    Resolves service.yourdomain.duckdns.org → NPM internal IP
        |
    NPM (handles HTTPS)
        |
    Nextcloud / Immich / Vaultwarden
```

### Away from home
```
Device
        |
    Tailscale (encrypted WireGuard tunnel)
        |
    Home network (subnet routed via Tailscale LXC)
        |
    NPM → services
```

No ports are open on the router. DuckDNS provides a domain name purely to obtain a valid Let's Encrypt TLS certificate, it doesn't route any traffic. Pi-hole resolves the domain internally to NPM's local IP so services work on the LAN without traffic leaving the network.

---

## Why No Port Forwarding

All services are kept internal by design. Nothing on the server is reachable from the internet without going through Tailscale's authenticated encrypted tunnel. Smaller attack surface, simpler to reason about.

---

## Storage

The laptop has a 512GB internal NVMe SSD. I had a spare 512GB Samsung external SSD so added it to Proxmox as a Directory storage pool named 'samsung'. The storage-heavy containers (Nextcloud and Immich) have their root disks on the samsung pool while lighter services stay on the internal drive.

---

## Proxmox Layout

| CT ID | Type | Service | RAM allocated | Storage Pool |
|---|---|---|---|---|
| 100 | LXC | Vaultwarden | 256MB | Internal |
| 101 | LXC | Nginx Proxy Manager | 512MB | Internal |
| 102 | LXC | Pi-hole | 512MB | Internal |
| 103 | LXC | Immich | 6GB | Samsung (external) |
| 104 | LXC | Nextcloud (NextcloudPi) | 6GB | Samsung (external) |
| 105 | LXC | Tailscale | 512MB | Internal |

This setup allows plenty of room for further additions


---

## Key Design Decisions

**LXC over VMs** — Used LXC containers rather than full VMs to reduce RAM and storage overhead on limited hardware.

**LVM-Thin over ZFS** — ZFS requires significant RAM (4GB+ minimum). On 16GB shared across multiple containers, LVM-Thin was the practical choice.

**NPM as single internal entry point** — All services sit behind NPM. TLS and routing managed in one place.

**DuckDNS for TLS only** — Free dynamic DNS used solely to get a valid Let's Encrypt certificate for HTTPS. No traffic routes through it.

**Tailscale with subnet routing** — Rather than installing Tailscale on every container, a dedicated Tailscale LXC advertises the entire home subnet. All containers are reachable via Tailscale without needing the client on each one.

**PVE Helper Scripts for Nextcloud and Immich** — Used [community Proxmox helper scripts](https://community-scripts.org/) to deploy these. Faster than a manual install, good tradeoff for a homelab.

---

## Problems Hit

**TLS certificates failing** — Let's Encrypt DNS challenge kept failing on first attempt due to DNS propagation delay. Fixed by adding 120 seconds to the propagation field in NPM and retrying.

**Internal DNS not resolving** — Devices couldn't reach services by domain name until local DNS records were added in Pi-hole pointing the DuckDNS subdomains to NPM's internal IP.

**Service config issues post-install** — Nextcloud needed trusted domain entries added to config.php. Vaultwarden and Immich both needed WebSocket support enabled in NPM.
