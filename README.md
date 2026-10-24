# homelab

This repo holds the Docker Compose stacks and configs for my homelab, so I can spin up my whole setup on a new machine in just a few clicks.

## Goal

I want every service I self-host (reverse proxy, apps, monitoring, etc.) defined as code in this repo. That way, rebuilding or migrating the homelab is just cloning this repo, dropping in secrets, and running `docker compose up`.

## Structure

Each top-level folder is a standalone stack with its own `docker-compose.yml` and `.env.example`:

- `traefik/`: Reverse proxy with automatic SSL via Let's Encrypt, using the Cloudflare DNS challenge.
- `media/`: Jellyfin + *arr stack (qBittorrent, SABnzbd, Prowlarr, Radarr, Sonarr, Seerr) for media management. See [`media/README.md`](media/README.md) for setup and the required data folder structure.
- `actual-budget/`: [Actual Budget](https://actualbudget.org/) server for keeping track of money.
- `duplicati/`: Backs up everyone else's config/state so I'm not starting from scratch if a drive dies.

Alongside those there are three shared, gitignored folders that aren't stacks themselves, just storage the stacks above point into:

- `data/`: media files and downloads (used by `media/`).
- `configs/`: each service's config/state, one subfolder per service, e.g. `configs/jellyfin` (used by `media/`, `actual-budget/`, `duplicati/`).
- `backups/`: where `duplicati/` writes its backups of `configs/`.

Every stack's `.env` points at these same three folders via `DATA_ROOT`, `CONFIG_ROOT`, and `BACKUP_ROOT`, so different stacks' containers see identical paths on disk and nothing needs remapping.

## Usage

First, create the shared folders (once, from the repo root):

```bash
mkdir -p data/downloads/qbittorrent/{completed,incomplete,torrents}
mkdir -p data/downloads/nzbget/{completed,intermediate,nzb,queue,tmp}
mkdir -p data/movies data/shows
mkdir -p configs backups
```

(Per-service folders under `configs/`, like `configs/jellyfin`, get created on their own the first time each stack runs.)

Then, for each stack:

```bash
cd <stack>
cp .env.example .env
# fill in hostnames, API keys, and double check the DATA_ROOT/CONFIG_ROOT/BACKUP_ROOT paths
docker compose up -d
```

The `.env.example` files already point `DATA_ROOT`/`CONFIG_ROOT`/`BACKUP_ROOT` at this repo's `data/`, `configs/`, and `backups/` folders, so if you cloned the repo to the same spot you shouldn't need to touch those — just the hostnames and keys. See [`media/README.md`](media/README.md) for a couple of media-stack-specific steps (qBittorrent's paths need a manual tweak after first login).

`.env` files, `data/`, `configs/`, `backups/`, and generated certs are gitignored and never committed.

## Status

Work in progress. I'll keep adding stacks here as I set them up.
