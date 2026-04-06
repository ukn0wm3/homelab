# Scripts

Install scripts used to set up services on this homelab. All scripts are official and sourced from their respective projects.

## Immich

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/immich.sh)"
```

Source: [Proxmox VE Community Scripts](https://community-scripts.org/scripts/immich)

## NextcloudPi

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/nextcloudpi.sh)"
```

Source: [Proxmox VE Community Scripts](https://community-scripts.org/scripts/nextcloudpi)

## Pi-hole

```bash
curl -sSL https://install.pi-hole.net | bash
```

Source: [Pi-hole](https://docs.pi-hole.net/main/basic-install/)

## Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Source: [Tailscale](https://tailscale.com/download/linux)

For further instructions view this [video](https://www.youtube.com/watch?v=JC63OGSzTQI)

## Vaultwarden & NPM

No scripts — both deployed manually via `docker-compose` with official images:
- `vaultwarden/server`
- `jc21/nginx-proxy-manager`