# Nextcloud (NextcloudPi)

## What it does

Nextcloud is a self-hosted file sync and sharing platform, essentially a personal Google Drive. It syncs files across devices via the Nextcloud desktop and mobile clients. All files stay on hardware I control rather than a third-party cloud provider.

## Deployment

- **Type:** LXC container (CT 103)
- **RAM:** 6GB
- **Storage:** 300GB on Samsung external SSD (container root disk)
- **Installed via:** [Proxmox VE Scripts](https://community-scripts.org/) — NextcloudPi variant

The helper script installs NextcloudPi, which comes with Apache, MariaDB, and PHP pre-configured. Significantly faster than a manual install(I tried manual, not worth the hours spent on troubleshooting).

See [`scripts/nextcloud-install.sh`](../scripts/nextcloud-install.sh) for the script used.

## External access

Accessible on the LAN via NPM and remotely via Tailscale. Not publicly exposed.

NPM proxy host points HTTPS → Nextcloud LXC on port 443.

## Storage

The container's root disk lives on the Samsung external SSD. This gives Nextcloud ~200GB to work with for files and application data without touching the internal drive.

## Config changes post-install

**Trusted domain** — After setting up the NPM proxy host, Nextcloud showed "Access through untrusted domain". Fixed by adding the DuckDNS domain to `trusted_domains` in `/var/www/nextcloud/config/config.php`.

**HTTPS protocol** — Nextcloud was serving internally over HTTP but needed to report HTTPS to clients. Fixed via occ commands:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:set overwriteprotocol --value='https'

# Nextcloud only sees HTTP from NPM internally. This tells it that clients are actually connecting over HTTPS so it stops generating http:// links

sudo -u www-data php /var/www/nextcloud/occ config:system:set overwrite.cli.url --value='https://nextcloud.yourdomain.duckdns.org'

# Sets the public address Nextcloud uses when generating links. Without this it falls back to the internal IP which breaks the mobile app login

# A better solution would be to change the NPM configuration rather then force overwrite
```

**Large file uploads** — Configured PHP 8.3 to allow large uploads by updating `/etc/php/8.3/fpm/php.ini`:
- `upload_max_filesize = 16G`
- `post_max_size = 16G`
- `memory_limit = -1`

Added timeout and body size overrides to the NPM advanced config for the proxy host.

## Problems Hit

**Trusted domain error** — Nextcloud blocks access from unrecognised domains by default. Adding the DuckDNS subdomain to `trusted_domains` in config.php fixed it.

**occ commands failing with RedisException** — occ must always be run as the www-data user, not root. Running as root causes session/cache errors.

**Large video uploads failing from iOS** — The iOS Nextcloud app requires ~ 4x the file size as free local storage before it will upload. Large video files failed silently. Immich is better suited for phone video backup.

## What I'd do differently

Use Immich for all photo and video backup from the start. Nextcloud is better suited for documents and files. Splitting the two responsibilities makes both work better.
