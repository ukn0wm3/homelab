# 🏠 Homelab

A self-hosted home server built on an old laptop running Proxmox, serving as a personal cloud, media backup, password manager, and network-level ad blocker. This repo documents my setup, the decisions I made, the problems I ran into, and the scripts I used.

> ⚠️ All credentials, domain names, and IP addresses have been replaced with placeholders.

---

## 🖥️ Hardware

| Component | Detail |
|---|---|
| Device | Old laptop (repurposed) |
| RAM | 16GB |
| Storage | 512GB internal SSD + 512GB external SSD |
| Hypervisor | Proxmox VE |

---

## 🧱 Services

| Service | Purpose | Deployed As |
|---|---|---|---|
| [Nginx Proxy Manager](nginx-proxy-manager.md) | Reverse proxy + TLS | LXC |
| [Pi-hole](pi-hole.md) | DNS + ad blocking | LXC | LAN only |
| [Nextcloud](nextcloud.md) | File storage + sync | LXC (PVE Helper Script) | 
| [Immich](immich.md) | Photo backup | LXC (PVE Helper Script) | 
| [Vaultwarden](vaultwarden.md) | Password manager | LXC | 
| [Tailscale](tailscale.md) | Remote access VPN | LXC | 

---

## 🌐 How Traffic Flows

All services are **internal only** — no ports are forwarded on the router. There are two ways to reach them:

**On the home network:**
```
Device → Pi-hole (DNS) → NPM (HTTPS) → Service
```

**Away from home:**
```
Device → Tailscale (encrypted tunnel) → Proxmox host → NPM → Service
```

DuckDNS is used purely to get a valid Let's Encrypt TLS certificate for HTTPS; it doesn't expose anything publicly. Pi-hole resolves the domain internally so it works on the LAN without hitting the internet.

---

## 📁 Files

```
homelab/
├── README.md
├── architecture.md
├── nginx-proxy-manager.md
├── pi-hole.md
├── nextcloud.md
├── immich.md
├── vaultwarden.md
├── tailscale.md
└── scripts/
    ├── README.md
    ├── nextcloud-install.sh
    └── immich-install.sh
```

---

## 🧠 Why I Built This

Wanted to move away from relying on third-party cloud services for personal data and learn practical self-hosting, networking, and Linux administration. Running this server has given me hands-on experience with:

- Proxmox virtualisation and LXC containers
- Reverse proxying and TLS certificate management
- DNS, split-horizon DNS, and local name resolution
- Docker and Docker Compose
- WireGuard-based networking via Tailscale
- Linux service administration and troubleshooting


