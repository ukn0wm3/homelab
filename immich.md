# Immich

## What it does

Immich is a self-hosted photo and video backup solution. A replacement for Google Photos. The mobile app automatically backs up photos from phones to the home server. It includes a web UI with facial recognition, album organisation, and a map view.

## Deployment

- **Type:** LXC container (CT 103)
- **RAM:** 6GB
- **Storage:** 300GB on Samsung external SSD (container root disk)
- **Installed via:** [Proxmox VE Community Scripts](https://community-scripts.github.io/ProxmoxVE/) (community maintained)

The helper script installs Immich as a native app managed by two systemd services — `immich-web.service` (API + web UI on port 2283) and `immich-ml.service` (machine learning). App files live at `/opt/immich/`.

```bash
# Managing Immich
systemctl restart immich-web.service immich-ml.service

# Logs
/var/log/immich/web.log
/var/log/immich/ml.log
```

See [`scripts/immich-install.sh`](../scripts/immich-install.sh) for the script used.

## External access

Accessible on the LAN via NPM and remotely via Tailscale. Not publicly exposed.

NPM proxy host points to Immich on port 2283 with WebSocket support enabled (required for the iOS app to show Server Online status).

## Storage

The container's root disk lives on the Samsung external SSD, giving Immich 300GB for the photo and video library. The media location is configured in `/opt/immich/.env`:

```
IMMICH_MEDIA_LOCATION=/mnt/immich-data
```

## Migrating existing photos — immich-go

To import existing photo libraries, used [immich-go](https://github.com/simulot/immich-go) — an open-source Go binary for bulk uploads. No Node.js required. Handles duplicate detection automatically via checksums, so importing from multiple sources is safe.

**Google Photos** (zips don't need extracting — immich-go reads them directly):
```bash
immich-go upload from-google-photos \
  --server=http://YOUR_IMMICH_IP:2283 \
  --api-key=YOUR_API_KEY \
  --sync-albums --include-unmatched \
  takeout-*.zip
```

**iCloud** (extract zip first):
```bash
immich-go upload from-icloud \
  --server=http://YOUR_IMMICH_IP:2283 \
  --api-key=YOUR_API_KEY \
  --manage-heic-jpeg=StackCoverJPG \
  /path/to/photos
```

**Local folder / Snapchat** (extract zips first):
```bash
immich-go upload from-folder \
  --server=http://YOUR_IMMICH_IP:2283 \
  --api-key=YOUR_API_KEY \
  --folder-as-album=FOLDER \
  /path/to/photos
```

Tip: always run with `--dry-run` first to see what will be imported before committing.

Result: ~3000 photos/videos imported across Google Photos, iCloud, and Snapchat with 0 errors. Duplicates between sources were detected and skipped automatically.

## Problems hit

**Immich stuck in maintenance mode for a long time on first install** — The helper script compiles native libraries from source on first run. On an older CPU this can take 15 minutes to 2 hours. If the screen does not progress after a long wait, try navigating to the login or admin page via the URL bar

**WebSocket errors in NPM** — Immich requires WebSocket support in the NPM proxy host for the iOS app to show Server Online status. Enabling it in NPM's advanced settings fixed it.

**immich-go 401 Unauthorized** — Caused by using `--api-key==` (double equals) instead of `--api-key=` (single equals).

## What I'd do differently

The initial install attempt used a manual Docker Compose approach which had compatibility issues and had to be scrapped. Using the community helper script from the start would have saved a significant amount of time.
