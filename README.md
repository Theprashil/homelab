# homelab

This repo holds the Docker Compose stacks and configs for my homelab, so I can spin up my whole setup on a new machine in just a few clicks.

## Goal

I want every service I self-host (reverse proxy, apps, monitoring, etc.) defined as code in this repo. That way, rebuilding or migrating the homelab is just cloning this repo, dropping in secrets, and running `docker compose up`.

## Structure

Each top-level folder is a standalone stack with its own `docker-compose.yml`.

- `traefik/`: Reverse proxy with automatic SSL via Let's Encrypt, using the Cloudflare DNS challenge.
- `media/`: Jellyfin + *arr stack (qBittorrent, SABnzbd, Prowlarr, Radarr, Sonarr, Seerr) for media management. See [`media/README.md`](media/README.md) for setup and the required data folder structure.

## Usage

Each stack has its own `.env.example`. Copy it to `.env` and fill in the required secrets before starting the stack:

```bash
cd <stack>
cp .env.example .env
docker compose up -d
```

`.env` files and generated certs are gitignored and never committed.

## Status

Work in progress. I'll keep adding stacks here as I set them up.
