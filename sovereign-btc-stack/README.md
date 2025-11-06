# Sovereign BTC Stack

A production-ready, modular Docker Compose stack for running a personal Bitcoin + Lightning server with:

- `bitcoind` (Bitcoin Core)
- `eclair` (ACINQ Lightning node)
- `electrs` (Electrum server)
- `nbxplorer` (Block explorer backend for BTCPay)
- `postgres` (Database for BTCPay)
- `btcpayserver` (Self-hosted payment processor)
- `RTL` (Ride the Lightning web UI)
- `traefik` (reverse proxy with automatic TLS via Let’s Encrypt)

This project aims for one-click deployment, secure defaults, and persistence via named Docker volumes.

## Features
- Private Docker bridge network (`sovereign-net`) isolates internal traffic
- Only necessary ports exposed externally; Traefik handles `80/443` TLS
- All user variables centralized in `.env` (copy from `.env.template`)
- Secrets protected via `.gitignore` to avoid committing `.env`
- Named volumes for chain and app data (`bitcoin_data`, `eclair_data`, `electrs_data`, `btcpay_data`)

## Prerequisites
- Docker Desktop (Engine 24+ and Compose v2)
- Public DNS records:
  - `A` record for `${DOMAIN_NAME}` → your server
  - Optional: `A` record for `rtl.${DOMAIN_NAME}` → your server
- Open inbound ports: `80`, `443`, `8333`, `9735` (optionally `50001` if Electrum is exposed)

## Quick Start
1) Copy and edit environment variables:
   - Windows PowerShell: `Copy-Item .env.template .env`
   - Fill values in `.env` (RPC creds, alias, domain, email, etc.)

2) Launch the stack:
   - `docker compose up -d`

3) Verify services:
   - Bitcoin P2P: `8333`
   - Lightning P2P: `9735`
   - Electrum TCP: `50001` (if you exposed it)
   - BTCPay Web: `https://<DOMAIN_NAME>`
   - RTL Web: `https://rtl.<DOMAIN_NAME>`

## Configuration Notes
- Config files in `./config/**` contain placeholders like `${BITCOIN_RPC_USER}` for readability and centralization.
- Services and env injection:
  - `bitcoind`: config mounted; uses `zmqpubhashblock` and segwit address types.
  - `eclair`: modern config via `JAVA_OPTS` (file optional).
  - `electrs`: configured via CLI flags; config file provided for reference.
  - `btcpayserver`: wired to `nbxplorer` and `postgres` via env.
  - `rtl`: config tokens are rendered from env on container start.

## Security
- `.env` is git-ignored; never commit real secrets.
- Use strong passwords for RPC/API.
- For internet exposure, ensure firewall rules only allow needed ports.
- TLS/ACME: handled by Traefik automatically with Let’s Encrypt for `DOMAIN_NAME` and optional `rtl.${DOMAIN_NAME}`.

## Persistence and Backup
- Named volumes keep data across restarts:
  - `bitcoin_data`, `eclair_data`, `electrs_data`, `btcpay_data`
- Back up volumes regularly (e.g., `docker run --rm -v <volume>:/data -v $PWD:/backup alpine tar -czf /backup/<volume>.tgz /data`).

## Operations
- Start: `docker compose up -d`
- Stop: `docker compose down`
- Logs: `docker compose logs -f <service>`
- Update images: `docker compose pull && docker compose up -d`

Health checks:
- `bitcoind` health uses `bitcoin-cli` RPC. If you change RPC creds, update `.env` accordingly.

Electrum considerations:
- If exposing `50001` publicly, consider TLS (`50002`) behind Traefik or nginx.

## Disclaimer
This is a minimal, modular baseline following your requested images and layout. Depending on your production needs, you may harden with:
- Reverse proxy + automatic TLS
- Metrics/alerts (Prometheus, Grafana)
- Automated backups
- CI/CD for image updates

Note: Recent Eclair releases require Bitcoin Core 29+. If your `bitcoind` image tag does not provide a recent enough version, switch to an image/tag that does, or build your own `bitcoind` container. The current compose uses a widely adopted image with `latest`, which may lag behind; pin to a compatible version where possible.