# Architecture

## Overview

The server is an old laptop running Proxmox VE as the hypervisor. All services run as either LXC containers or inside a Docker Compose stack within an LXC, keeping them isolated and easy to manage independently.

All services are **internal-only** — no ports are forwarded on the router. Remote access is handled entirely by Tailscale.

---

## How Access Works

### On the home network (LAN)
```
Device (phone/laptop on WiFi)
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
Device (phone/laptop anywhere)
        |
    Tailscale (encrypted WireGuard tunnel)
        |
    Proxmox host on home network
        |
    NPM → services
```

No ports are open on the router. DuckDNS provides a domain name purely to obtain a valid Let's Encrypt TLS certificate. It doesn't route any traffic. Pi-hole resolves the domain internally to NPM's local IP so services work on the LAN without traffic leaving the network.

---

## Why No Port Forwarding

All services are kept internal by design. Nothing on the server is reachable from the internet without going through Tailscale's authenticated encrypted tunnel. Smaller attack surface, simpler to reason about.

---

## Storage

The laptop has a 512GB internal SSD. I had a spare 512GB external SSD so added it as a second storage pool in Proxmox making it 1TB total. Storage-heavy services (Nextcloud, Immich) are pointed at the external drive, keeping the OS and lightweight services on the internal one.

---

## Proxmox Layout

| ID | Type | Service | RAM allocated | Storage allocated |
|---|---|---|---|---|
| 100 | LXC | VaultWarden | 256MB | 4GB |
| 101 | LXC | Nginx Proxy Manager | 512MB | 4GB |
| 102 | LXC | Pi-hole | 512MB | 4GB |
| 103 | LXC | Immich  | 6GB | 300GB |
| 104 | LXC | Nextcloud | 6GB | 100GB |
| 105 | LXC | Tailscale | 512MB | 2GB |

This setup leaves plenty of room for additional services
---

## Key Design Decisions

**LXC over VMs** — Used LXC containers rather than full VMs to reduce RAM and storage overhead on limited hardware.

**NPM as single internal entry point** — All services sit behind NPM. TLS and routing managed in one place.

**DuckDNS for TLS only** — Free dynamic DNS used solely to get a valid Let's Encrypt certificate for HTTPS. No traffic routes through it.

**Tailscale for remote access** — Secure remote access with no public exposure. Only authenticated tailnet devices can reach the server.

**PVE Helper Scripts for Nextcloud and Immich** — Used [community Proxmox helper scripts](https://community-scripts.org/) to deploy these. Faster than a manual install, good tradeoff for a homelab.

---

## Problems Hit

**TLS certificates failing** — Let's Encrypt kept failing initially. Took some digging to understand it needed outbound internet access from the NPM container, not inbound port forwarding. Fixed once that was clear.

**Internal DNS not resolving** — Devices couldn't reach services by domain name until local DNS records were added in Pi-hole pointing the DuckDNS subdomains to NPM's internal IP (split-horizon DNS).

**Service config issues post-install** — Nextcloud needed trusted domain and reverse proxy entries added to config.php. Vaultwarden and Immich both needed WebSocket support enabled in NPM.
