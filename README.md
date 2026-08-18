# homelab

Sanitized reference configs for my self-hosted infrastructure stack. These are the actual configs I run — stripped of personal details and offered as a starting point for anyone building something similar.

## Stack Overview

| Component | Role | Host |
|-----------|------|------|
| TrueNAS SCALE | NAS + Docker host | Dell PowerEdge R710 |
| Proxmox | Hypervisor | Asus ROG Rampage IV |
| Traefik v3 | Reverse proxy + SSL | Docker on TrueNAS |
| Passbolt CE | Password manager | Docker on TrueNAS |
| Pi-hole v6 + Unbound | DNS + ad blocking | Raspberry Pi 5 |
| WireGuard | VPN | Raspberry Pi 5 |
| Uptime Kuma | Monitoring | Raspberry Pi 3 |
| Immich | Photo management | Docker on TrueNAS |
| Jellyfin | Media server | Docker on TrueNAS |
| ARM | Disc ripping pipeline | LXC on Proxmox |

## Repo Structure

```
homelab/
├── traefik/          # Traefik static config + file provider for non-Docker services
├── stacks/
│   └── passbolt/     # Traefik + Passbolt CE docker-compose stack
└── scripts/          # Utility scripts (Cloudflare DDNS, etc.)
```

## Design Principles

- **Bind mounts** over named volumes for anything I care about — data lives where I can see it
- **`.env` files** per stack — no secrets in compose files, no variable interpolation inside `.env`
- **Traefik file provider** for non-Docker services (Proxmox, TrueNAS, ARM) with hot-reload enabled
- **Cloudflare DNS challenge** for wildcard certs — no port 80 exposure required
- **`docker compose down && up`** (not `restart`) to apply new volume mounts or env var changes

## Traefik Notes

- File provider: any invalid field name causes the **entire file** to be rejected — all file-provider routes go down simultaneously. Docker-label routes are unaffected. Validate carefully.
- `docker logs traefik` produces **no output** — logs write to file only (configured in traefik.yml)
- Multi-network containers need the `traefik.docker.network=proxy` label
- `insecureSkipVerify: true` via a named `serversTransport` for upstream services with self-signed certs (Proxmox, TrueNAS)

## Related Repos

- [`dotfiles`](https://github.com/LordMortus/dotfiles) — workstation configs (zsh, niri, alacritty, waybar, tmux)
