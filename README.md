# data

S3 file server powered by [MinIO AIStor](https://www.min.io/product/aistor) (free-tier license), deployed with Docker Compose behind an existing [Traefik](https://traefik.io/) reverse proxy.

## Overview

A single MinIO AIStor container exposes two ports. Both are served through Traefik on the same HTTPS `:443` entrypoint and routed by hostname:

| Hostname                     | Container port | Purpose          |
| ---------------------------- | -------------- | ---------------- |
| `data.simovilab.org`         | 9000           | S3 API           |
| `console.data.simovilab.org` | 9001           | Console (web UI) |

Traefik terminates TLS; MinIO speaks plain HTTP inside the `traefik-proxy` network, so no certificates are mounted into the container.

## Prerequisites

- A running Traefik instance with:
  - a Docker network named `traefik-proxy`,
  - an HTTPS entrypoint named `websecure` (`:443`),
  - a certificate resolver named `letsencrypt`.
- DNS records for `data.simovilab.org` **and** `console.data.simovilab.org` pointing at the server.
- A MinIO AIStor license saved as `minio.license` in this directory (free-tier license available from the [MinIO pricing page](https://min.io/pricing)).

## Setup

1. Create your local config from the template:
   ```bash
   cp .env.example .env
   ```
   Set a strong `MINIO_ROOT_PASSWORD` (e.g. `openssl rand -base64 24`) and adjust the domains or Traefik names if your setup differs.
2. Make sure `minio.license` is present in this directory.
3. Start the server:
   ```bash
   docker compose up -d
   docker compose logs -f minio
   ```

## Access

- **S3 API:** `https://data.simovilab.org`
- **Console:** `https://console.data.simovilab.org`

Connect with the AIStor client (`mc`):

```bash
mc alias set data https://data.simovilab.org "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD"
mc mb data/my-bucket
mc cp ./somefile data/my-bucket
```

## Notes

- TLS is handled entirely by Traefik. The `certs` directory from the MinIO docs is not needed in this setup.
- The Console is pinned to port `9001` via `--console-address`; `MINIO_BROWSER_REDIRECT_URL` tells it its public URL so logins and redirects work behind the proxy.
- `.env` and `minio.license` are gitignored — keep them out of version control.
- For production, pin a specific `RELEASE.*` image tag in `.env` instead of `:latest`.
