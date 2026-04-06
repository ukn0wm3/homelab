# Tailscale

## What it does

Tailscale creates a private encrypted network (called a tailnet) between all devices such as phone, laptop, and home server regardless of where they are. Services that are only accessible on the home LAN become reachable securely from anywhere, without exposing any ports to the internet.

Built on top of WireGuard. Tailscale handles key exchange and NAT traversal automatically.

## Why I use it

All self-hosted services are internal-only, no ports are forwarded on the router. Tailscale is how they're accessed remotely. It's a much cleaner approach than opening ports.

## Deployment

- **Type:**  LXC container (CT 105)
- **RAM:** 512MB
- **Storage:** Internal SSD
- **Installed via:** Official Tailscale one-liner script

After install, running `tailscale up` outputs an authentication URL which you open in a browser on another device to connect the LXC to the tailnet.
## Subnet routing

Rather than installing Tailscale on every container individually, the Tailscale LXC advertises the entire home subnet. This means every container on the network is reachable via Tailscale without any additional setup on each one.

The subnet route was approved in the Tailscale admin console after being advertised.

## DNS integration

Pi-hole is set as the global DNS nameserver in the Tailscale admin console with Override DNS enabled. This means when Tailscale is active, all DNS queries route through Pi-hole. This makes all internal service names resolve correctly from anywhere, exactly as they do on the home network.

Cloudflare (1.1.1.1) is set as a fallback nameserver in case Pi-hole goes down.

## How it works in practice

- **Tailscale ON + away from home** — DNS routes through Pi-hole, internal domain names resolve, all services accessible
- **Tailscale OFF** — Normal internet works, no access to home services
- **Not configured as an exit node** — only DNS and LAN access route through home, not all internet traffic

## Problems Hit

**Domain names not resolving on ethernet** — DNS was only configured on the WiFi adapter on Windows. Setting it on the ethernet adapter too fixed resolution when plugged in via cable.

## Security

- Zero ports exposed on the router
- Only devices authenticated to the Tailscale account can join the tailnet
- Traffic encrypted end-to-end via WireGuard