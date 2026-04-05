# Immich

## What it does

Immich is a self-hosted photo and video backup solution — a replacement for Google Photos. The mobile app automatically backs up photos from phones to the home server. It includes a web UI with facial recognition, album organisation, and a map view.

## Deployment

- **Type:** LXC container running Docker Compose
- **RAM:** 4GB
- **Storage:** External SSD
- **Installed via:** [Proxmox VE Helper Scripts](https://tteck.github.io/Proxmox/) (community maintained)

Immich runs as multiple services (server, microservices, machine learning, Redis, PostgreSQL) coordinated via Docker Compose. The helper script sets up the LXC with Docker and deploys the full stack.

See [`scripts/immich-install.sh`](../scripts/immich-install.sh) for the script used.

## External access

Accessible on the LAN via NPM and remotely via Tailscale. Not publicly exposed.

## Storage

Photo library volume mounted from the external SSD. I had a spare 512GB external drive so pointed Immich's data directory at it — keeps the OS and app on the internal drive while photos live separately, which is cleaner anyway.

## Migrating existing photos — immich-go

To import an existing photo collection into Immich, I used [immich-go](https://github.com/simulot/immich-go) — an open-source Go binary that handles bulk uploads without needing Node.js installed.

It supports uploading from local folders, Google Photos Takeout, iCloud exports, and more. It also handles duplicate detection so photos don't get imported twice.

```sh
# Upload from a local folder
immich-go upload from-folder \
  --server=http://YOUR_IMMICH_IP:2283 \
  --api-key=YOUR_API_KEY \
  /path/to/photos
```

The API key is generated in Immich's web UI under user settings.

## Resource usage

The machine learning container (facial recognition and CLIP search) is the most RAM-intensive component. The LXC is set to 4GB total, with the ML container limited to prevent it starving other services.

## Problems Hit

**NPM proxy configuration** — Immich requires WebSocket support for the web UI to work correctly. Had to enable the WebSocket option in the NPM proxy host settings.

**WebSocket errors in web UI** — Some Immich features use WebSockets for real-time updates. Enabling WebSocket support in NPM fixed this.

## What I'd do differently

Set up the external storage mount before deploying rather than configuring it after — it's cleaner to point the volume at the right place from the start.
