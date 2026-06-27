# Self-Hosted Media Stack

A Docker Compose stack for a home media server built around Jellyfin and the Arr automation ecosystem. It includes download management, indexer management, media library automation, subtitle automation, request management, and streaming.

This repository is intended to store only Compose configuration. Runtime data, application config, downloads, and media libraries are mounted from host paths under `/data` and should not be committed.

## Services

| Service | Port | Purpose |
|---|---:|---|
| `jellyfin` | `8096` | Media streaming server for movies, TV, and music. |
| `qbittorrent` | `8080` | Download client Web UI. |
| `prowlarr` | `9696` | Indexer manager for Radarr/Sonarr/Lidarr. |
| `radarr` | `7878` | Movie library automation. |
| `sonarr` | `8989` | TV series library automation. |
| `lidarr` | `8686` | Music library automation. |
| `bazarr` | `6767` | Subtitle automation for movies and TV. |
| `seerr` | `5055` | Media request and discovery UI. |

## Directory Layout

The stack expects these host paths:

```text
/data/config/qbittorrent
/data/config/prowlarr
/data/config/radarr
/data/config/sonarr
/data/config/lidarr
/data/config/seerr
/data/config/bazarr
/data/config/jellyfin
/data/downloads/complete
/data/downloads/incomplete
/data/media/movies
/data/media/tv
/data/media/music
/data/media
```

Container mounts:

| Host Path | Container Path | Used By |
|---|---|---|
| `/data` | `/data` | qBittorrent, Radarr, Sonarr, Lidarr, Bazarr |
| `/data/downloads` | `/downloads` | qBittorrent only, compatibility for old torrents |
| `/data/media` | `/media` | Jellyfin |
| `/data/config/<app>` | app-specific config path | each service |

Radarr, Sonarr, Lidarr, Bazarr, and qBittorrent intentionally share the same `/data` mount. This keeps downloads and media libraries on the same visible filesystem path inside the containers, so Arr imports can use hardlinks instead of creating duplicate full-size copies.

qBittorrent also keeps a compatibility `/downloads` mount for torrents that were added before the migration to `/data`. New downloads should still use `/data/downloads/complete` and `/data/downloads/incomplete`.

## Requirements

- Docker Engine with Docker Compose.
- Linux host or server with persistent storage mounted at `/data`.
- A user/group id for file ownership, configured through `PUID` and `PGID`.

## Quick Start

1. Copy the environment template:

   ```bash
   cp .env.example .env
   ```

2. Edit `.env`:

   ```env
   PUID=1000
   PGID=1000
   TZ=Asia/Jakarta
   ```

3. Create host directories:

   ```bash
   sudo mkdir -p \
     /data/config/qbittorrent \
     /data/config/prowlarr \
     /data/config/radarr \
     /data/config/sonarr \
     /data/config/lidarr \
     /data/config/seerr \
     /data/config/bazarr \
     /data/config/jellyfin \
     /data/downloads/complete \
     /data/downloads/incomplete \
     /data/media/movies \
     /data/media/tv \
     /data/media/music
   ```

4. Start the stack:

   ```bash
   docker compose up -d
   ```

5. Open the apps:

   ```text
   Jellyfin:     http://<host>:8096
   qBittorrent:  http://<host>:8080
   Prowlarr:     http://<host>:9696
   Radarr:       http://<host>:7878
   Sonarr:       http://<host>:8989
   Lidarr:       http://<host>:8686
   Bazarr:       http://<host>:6767
   Seerr:        http://<host>:5055
   ```

## First-Time Setup Notes

### qBittorrent

- Set the completed download path to `/data/downloads/complete`.
- Set the incomplete download path to `/data/downloads/incomplete`.
- Connect Radarr/Sonarr/Lidarr to qBittorrent using the Docker service/container host if they share a network, or the host IP if configured externally.

### Prowlarr

- Add indexers in Prowlarr.
- Connect Prowlarr to Radarr, Sonarr, and Lidarr.

### Radarr

- Root folder: `/data/media/movies`.
- Download client category recommendation: `radarr`.

### Sonarr

- Root folder: `/data/media/tv`.
- Download client category recommendation: `sonarr`.

### Lidarr

- Root folder: `/data/media/music`.
- Download client category recommendation: `lidarr`.

### Bazarr

- Connect Bazarr to Radarr and Sonarr.
- Movies path: `/data/media/movies`.
- TV path: `/data/media/tv`.

### Jellyfin

- Add media libraries from `/media`, or more specific paths inside it:
  - `/media/movies`
  - `/media/tv`
  - `/media/music`

## Common Commands

Start or update the stack:

```bash
docker compose up -d
```

View running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Restart one service:

```bash
docker compose restart jellyfin
```

Pull latest images and recreate:

```bash
docker compose pull
docker compose up -d
```

Stop the stack:

```bash
docker compose down
```

## Public Repository Notes

- Do not commit `.env`.
- Do not commit app config from `/data/config`.
- Do not commit media files or downloads.
- Use `.env.example` for documented defaults.
- Be careful with public ports. Prefer reverse proxy, VPN, or Tailscale for remote access.

## Security Notes

- qBittorrent and Arr apps should not be exposed directly to the public internet.
- Put services behind a reverse proxy with authentication if remote access is required.
- Keep application credentials, API keys, and indexer credentials inside app config volumes, not in this repository.

## Repository Name Ideas

Good names for this stack:

- `selfhosted-media-stack`
- `homelab-media-stack`
- `docker-media-server`
- `jellyfin-arr-stack`
- `jalu-media-stack`

Recommended: **`homelab-media-stack`**. It is clear, short, and broad enough for Jellyfin, qBittorrent, Seerr, and the Arr services.
