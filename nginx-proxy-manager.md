# Nginx Proxy Manager

## What it does

NPM acts as the single entry point for all HTTPS traffic on the home network. It receives requests from devices on the LAN (and via Tailscale), terminates TLS, and routes to the correct service based on the subdomain.

It also handles automatic Let's Encrypt certificate issuance and renewal via the DuckDNS DNS challenge. The certificate is what enables proper HTTPS. Without it, browsers show warnings and Vaultwarden refuses to work entirely.

## Deployment

- **Type:** LXC container (CT 101)
- **RAM:** 512MB
- **Storage:** Internal SSD
- **Installed via:** Docker Compose inside the LXC (`apt install docker.io docker-compose`)

## Proxy Hosts

| Domain | Forwarded To | WebSockets |
|---|---|---|
| vault.yourdomain.duckdns.org | Vaultwarden. IP:80 | Enabled |
| nextcloud.yourdomain.duckdns.org | Nextcloud. IP:443 | — |
| immich.yourdomain.duckdns.org | Immich. IP:2283 | Enabled |

## How the cert works without port forwarding

Let's Encrypt normally validates via an HTTP challenge . It tries to reach your server from the internet. Since no ports are forwarded this would fail.

Instead NPM uses the **DNS-01 challenge** via DuckDNS. NPM proves domain ownership by writing a temporary TXT record to DuckDNS rather than serving a file over HTTP. This was done by providing the API provided upon creation of the domain. No inbound ports needed.

A wildcard certificate (`*.yourdomain.duckdns.org`) was issued. This covers all subdomains with a single cert.

## Problems Hit

**SSL certificate save failed on first attempt** — Let's Encrypt DNS challenge failed due to DNS propagation delay. Fixed by setting the Propagation Seconds field to 120 in NPM and retrying after a few minutes.

**Internal DNS not resolving the domain** — After setting up proxy hosts, devices on the LAN couldn't reach services by domain name. Fixed by adding local DNS records in Pi-hole pointing each subdomain directly to NPM's internal IP.

**WebSocket errors** — Immich and Vaultwarden both require WebSocket connections for certain features. Had to enable WebSocket support in each proxy host's settings in NPM.

## What I'd do differently

Look into Cloudflare tunnels as an alternative; they also avoid port forwarding but add DDoS protection and don't require a DuckDNS account. Slightly more setup but more robust long term.
