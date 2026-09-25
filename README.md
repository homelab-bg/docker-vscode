# VS Code (code-server)

[linuxserver/code-server](https://docs.linuxserver.io/images/docker-code-server/) - VS Code in the browser, with support for standalone or Traefik reverse proxy deployments. The workspace mounts this stack's own compose files and sibling app directories (`../dns-stack`, `../traefik-portainer`), so other stacks can be edited from the browser.

## Quick Start

1. Copy environment template:
```bash
cp .env.example .env
```

2. Edit `.env` - set `PUID`/`PGID` to the owner of the mounted app directories, and `VSCODE_HOST` if using Traefik.

`.env` must exist before the first deploy - it's bind-mounted into the workspace, and Docker creates a missing bind source as an empty directory.

## Deployment Options

### Standalone Deployment
Direct access via host port:
```bash
docker compose up -d
```
Access at: http://localhost:8443

### Traefik Integration
Deploy with reverse proxy integration on a single host:
```bash
docker compose -f docker-compose.yml -f docker-compose.traefik.yml up -d
```
Access at: https://your-domain (with automatic TLS)

## TrueNAS Deployment

Put this repo in a dataset alongside your other apps (e.g. `/mnt/<pool>/Apps/vscode`), then **Apps → Discover Apps → Install via YAML**:
```yaml
include:
  - path:
      - /mnt/<pool>/Apps/vscode/docker-compose.yml
      - /mnt/<pool>/Apps/vscode/docker-compose.traefik.yml

# Web Portal button in the TrueNAS Apps UI. This goes here, not in the compose
# files, because `include:` drops top-level x- extensions from included files.
# Use a literal hostname: .env isn't applied to this file.
x-portals:
  - {name: VS Code, scheme: https, host: vscode.yourdomain.com, port: 443, path: /}
```

The dataset is the project directory, so `.env`, `./config` and the `../<app>` workspace mounts all resolve under `/mnt/<pool>/Apps/`.

Set `PUID`/`PGID` in `.env` to the user that owns the app datasets, so files edited in vscode keep their ownership. The defaults are 1000/1000, but on TrueNAS that's usually `truenas_admin`:
```bash
PUID=950
PGID=950
```
To check the IDs on your system, run `id truenas_admin` or `ls -ln /mnt/<pool>/Apps`.

## Prerequisites

### Standalone
- Docker & Docker Compose

### Traefik Integration
- Docker & Docker Compose
- External `traefik` network
- Traefik instance with Let's Encrypt configured (e.g. [docker-traefik-portainer](https://github.com/homelab-bg/docker-traefik-portainer))

## Security

⚠️ No authentication is configured - `PASSWORD`/`HASHED_PASSWORD` are commented out, so anyone who can reach the host/port gets a terminal and read/write access to every mounted app directory (including other stacks' `secrets/`). Keep it LAN-only until auth is in place (e.g. Authentik forward-auth via Traefik).

## Configuration

See `.env.example` for available environment variables.
