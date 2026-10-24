# Media

Self-hosted media stack: Jellyfin for streaming, qBittorrent and SABnzbd for downloading, and the *arr trio (Prowlarr, Radarr, Sonarr) plus Seerr for automated library management and requests.

## Services

- **Jellyfin** — media server / streaming (`JELLYFIN_HOSTNAME`)
- **qBittorrent** — torrent downloads (`QBITTORRENT_HOSTNAME`)
- **SABnzbd** — usenet downloads (`SABNZBD_HOSTNAME`)
- **Prowlarr** — indexer manager, feeds Radarr/Sonarr (`PROWLARR_HOSTNAME`)
- **Radarr** — movie library automation (`RADARR_HOSTNAME`)
- **Sonarr** — TV show library automation (`SONARR_HOSTNAME`)
- **Seerr** — media requests (`SEERR_HOSTNAME`)

They all share `DATA_ROOT` (mounted as `/data` in every container) and `CONFIG_ROOT` (each service gets its own `${CONFIG_ROOT}/<service>` mounted as `/config`) — both set in the root [`README.md`](../README.md). Because every container sees the same `/data`, a path set inside one app's UI (e.g. qBittorrent's save path) is the exact same path Radarr/Sonarr will see when they pick up a finished download — no remote path mappings needed, as long as everything stays under `DATA_ROOT`.

## Setup

1. Copy the env file and fill in your values (hostnames, API keys, timezone, etc.):

   ```bash
   cp .env.example .env
   ```

2. Make sure the data folders exist (see root `README.md` for the one-liner that sets these up). They live under the repo's `data/`, which is gitignored, so they won't exist right after cloning:

   ```
   data/
   ├── downloads/
   │   ├── qbittorrent/
   │   │   ├── completed/
   │   │   ├── incomplete/
   │   │   └── torrents/
   │   └── nzbget/         # used by the sabnzbd service
   │       ├── completed/
   │       ├── intermediate/
   │       ├── nzb/
   │       ├── queue/
   │       └── tmp/
   ├── movies/
   └── shows/
   ```

3. Start the stack:

   ```bash
   docker compose up -d
   ```

## Post-setup: qBittorrent paths

qBittorrent's default download paths don't line up with this layout out of the box. After first login, go to **Settings → Downloads** and set:

- **Default Save Path:** `/data/downloads/qbittorrent/completed`
- **Keep incomplete torrents in:** `/data/downloads/qbittorrent/incomplete`
- **Copy .torrent files to:** `/data/downloads/qbittorrent/torrents`

These are container paths (everything under `${DATA_ROOT}` is mounted at `/data`), so when wiring qBittorrent up as a download client in Radarr/Sonarr, point them at the same `/data/downloads/qbittorrent/completed` path — since they share the same mount, no path remapping is required.
