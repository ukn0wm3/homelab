# Vaultwarden

## What it does

Vaultwarden is an unofficial, lightweight server implementation of Bitwarden. It lets me run my own password manager backend that the official Bitwarden apps and browser extensions connect to. This lets me keep all my passwords on hardware I own rather than Bitwarden's cloud.

It was set up first, before any other service, so that all passwords created during the rest of the homelab setup could be saved immediately.

## Deployment

- **Type:** LXC container (CT 100)
- **RAM:** 512MB
- **Storage:** Internal SSD
- **Installed via:** Docker inside the LXC (`apt install docker.io docker-compose`, then official `vaultwarden/server` image)

The community install script that most guides reference returned a 404 — the repo no longer exists. Used Docker directly instead.

## External access

Accessible on the LAN via NPM and remotely via Tailscale. Vaultwarden requires HTTPS to function. The Bitwarden clients will not connect over plain HTTP. This was the original reason for setting up NPM and DuckDNS.

## Security configuration

- **Signups disabled** — `SIGNUPS_ALLOWED=false` set after creating accounts. Prevents anyone who finds the URL from registering.
- **Admin panel disabled** — Used during initial setup then disabled by unsetting `ADMIN_TOKEN`.
- **2FA enabled** — TOTP two-factor authentication on all accounts.

## Problems Hit

**Vaultwarden refused to work over HTTP** — The error was "Insecure URL not allowed. All URLs must use HTTPS." This is what prompted setting up NPM and DuckDNS with a Let's Encrypt certificate before anything else.

**Community install script returned 404** — The script referenced in the guide no longer exists. Switched to running the official Docker image directly.

**docker compose (space) vs docker-compose (hyphen)** — The newer `docker compose` plugin syntax didn't work on Debian. Debian's apt installs the older standalone version which uses `docker-compose` with a hyphen.

**Domain not resolving before Pi-hole was set up** — Temporarily added an entry to the Windows hosts file as a workaround until Pi-hole local DNS was configured.

## What I'd do differently

Look into restricting access further either by limiting the NPM proxy host to Tailscale IP ranges only. It's the highest-value target in the stack.