# Nginx Proxy Manager

## What it does

NPM acts as the single entry point for all internal HTTPS traffic. It receives requests from devices on the local network (and via Tailscale), terminates TLS, and routes traffic to the correct service based on the subdomain.

It also handles automatic Let's Encrypt certificate issuance and renewal for the DuckDNS domain. The cert is what enables proper HTTPS — without it, browsers would show certificate warnings, and Vaultwarden would refuse to work entirely.

## Deployment

- **Type:** LXC container
- **RAM:** 512MB
- **Storage:** 8GB (internal SSD)

## Proxy Hosts

| Domain | Forwarded To | TLS |
|---|---|---|
| nextcloud.yourdomain.duckdns.org | Nextcloud LXC IP:80 | Let's Encrypt |
| photos.yourdomain.duckdns.org | Immich LXC IP:2283 | Let's Encrypt |
| vault.yourdomain.duckdns.org | Vaultwarden LXC IP:80 | Let's Encrypt |

## How the cert works without port forwarding

Let's Encrypt normally validates domain ownership via an HTTP challenge — it tries to reach your server from the internet. Since no ports are forwarded, this would fail.

Instead, NPM uses the **DNS-01 challenge** via DuckDNS. DuckDNS supports this natively — NPM proves domain ownership by writing a temporary DNS record rather than serving a file over HTTP. No inbound ports needed.

## Problems Hit

**Internal DNS not resolving the domain** — After setting up proxy hosts, devices on the local network couldn't reach services by domain name. The public DuckDNS DNS pointed to the home's external IP, and NAT loopback wasn't working reliably on the router. Fixed by adding local DNS records in Pi-hole pointing the subdomains directly to NPM's internal IP (split-horizon DNS).

**WebSocket errors** — Both Immich and Vaultwarden need WebSocket connections for certain features. Had to enable the WebSocket option in each NPM proxy host's advanced settings.

## What I'd do differently

Look into Cloudflare tunnels as an alternative — they also avoid port forwarding but add Cloudflare's DDoS protection and don't require a DuckDNS account. Slightly more complex to set up initially.
