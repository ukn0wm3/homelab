# Pi-hole + Unbound

## What it does

Pi-hole is a network-wide DNS sinkhole. It runs as the DNS server for all devices on the network, intercepting DNS queries and blocking ones that match known ad, tracker, and malware domains.

Beyond ad blocking, it handles custom local DNS records. I used it here to make internal services resolve correctly by domain name rather than IP.

**Unbound** runs alongside Pi-hole as a recursive DNS resolver. Rather than forwarding queries to a third-party upstream like Cloudflare or Google, Unbound resolves domains directly by querying the root DNS servers itself. Pi-hole points its upstream DNS at Unbound on port 5335, keeping DNS resolution entirely local and private.

## Deployment

- **Type:** LXC container (CT 102)
- **RAM:** 512MB
- **Storage:** Internal SSD
- **Pi-hole installed via:** Official curl installer
- **Unbound installed via:** apt

## Configuration

**Upstream DNS:** Unbound on 127.0.0.1#5335 (recursive, queries root servers directly)

**Unbound config location:** `/etc/unbound/unbound.conf.d/pi-hole.conf`

Unbound listens on 127.0.0.1 port 5335; same machine as Pi-hole, internal communication only. Root hints file downloaded from internic.net.

**Local DNS records added:**
- `vault.yourdomain.duckdns.org` → NPM IP
- `nextcloud.yourdomain.duckdns.org` → NPM IP
- `immich.yourdomain.duckdns.org` → NPM IP

All subdomains point to NPM's IP, not the individual service IPs. NPM then routes to the correct service.

## DNS Setup

DNS was set manually on each device individually rather than at the router level for Windows (both WiFi and Ethernet adapters), iOS, and Android. If your router allows changing the DNS server, you can set it there once and all devices will pick it up automatically via DHCP.

## Problems Hit

**Pi-hole doesn't support wildcard DNS**. I tried to add `*.yourdomain.duckdns.org` as a single catch-all record but Pi-hole rejected it. Each service subdomain has to be added individually as new services are created.

**DNS still resolving incorrectly after changes**. Windows was caching old DNS entries. Fixed by running `ipconfig /flushdns` after making any DNS changes.


## What I'd do differently

Set up a secondary Pi-hole instance so DNS doesn't go down if the primary LXC needs maintenance. Can run on very little RAM.
