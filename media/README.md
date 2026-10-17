# Media

Self-hosted media stack: Jellyfin for streaming, qBittorrent and SABnzbd for downloading, and the *arr trio (Prowlarr, Radarr, Sonarr) plus Seerr for automated library management and requests.

## Services

| Service     | Purpose                                      | Hostname env var       |
|-------------|-----------------------------------------------|-------------------------|
| Jellyfin    | Media server / streaming                      | `JELLYFIN_HOSTNAME`     |
| qBittorrent | Torrent downloads                             | `QBITTORRENT_HOSTNAME`  |
| SABnzbd     | Usenet downloads                              | `SABNZBD_HOSTNAME`      |
| Prowlarr    | Indexer manager, feeds Radarr/Sonarr           | `PROWLARR_HOSTNAME`     |
| Radarr      | Movie library automation                      | `RADARR_HOSTNAME`       |
| Sonarr      | TV show library automation                    | `SONARR_HOSTNAME`       |
| Seerr       | Media request management                      | `SEERR_HOSTNAME`        |

Every service shares one `DATA_ROOT` env var, which is bind-mounted into each container as `/data`. Configs for each app live under `${DATA_ROOT}/configs/<service>`, and media/downloads live directly under `${DATA_ROOT}`. Because every container sees the same `/data`, a path set inside one app's UI (e.g. qBittorrent's save path) is the exact same path Radarr/Sonarr will see when they pick up a finished download — no remote path mappings needed, as long as everything stays under `DATA_ROOT`.

## Setup

1. Copy the env file and fill in your values (`DATA_ROOT`, hostnames, API keys, timezone, etc.):

   ```bash
   cp .env.example .env
   ```

2. Create the data folder structure on the host. These folders live under `data/` (gitignored — see root `.gitignore`), so they won't exist after a fresh clone and need to be created manually before first run:

   ```bash
   mkdir -p data/{downloads/qbittorrent/{completed,incomplete,torrents},movies,shows}
   mkdir -p data/downloads/nzbget/{completed,intermediate,nzb,queue,tmp}
   ```

   Resulting layout:

   ```
   data/
   ├── configs/            # per-service app config/state, created automatically on first run
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
