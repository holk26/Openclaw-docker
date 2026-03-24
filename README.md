# Openclaw-docker

A Docker image that packages [OpenClaw](https://github.com/openclaw/openclaw) together with all its required tools, including Bun, Homebrew (Linuxbrew), pnpm, and Playwright/Chromium.

## Requirements

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2+ (optional, for `docker-compose.yml`)

## Building the image

Build from the latest `main` branch of OpenClaw (default):

```bash
docker build -t openclaw:latest .
```

Build from a specific branch or tag:

```bash
docker build --build-arg OPENCLAW_VERSION=<branch-or-tag> -t openclaw:latest .
```

## Running

Show available CLI options:

```bash
docker run --rm openclaw:latest --help
```

Run with a persistent workspace using Docker Compose:

```bash
docker compose up
```

The workspace directory is persisted in a Docker volume (`openclaw-workspace`) mounted at `/home/node/.openclaw/workspace` inside the container.

## Image details

| Component | Details |
|-----------|---------|
| Base image | `node:22-bookworm` |
| Package manager | pnpm (via corepack) |
| Build runtime | Bun |
| Homebrew (Linuxbrew) | Installed for first-party skills |
| Browser automation | Playwright + Chromium |
| Runtime user | `node` (non-root) |

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `production` | Node.js environment |
| `HOMEBREW_NO_AUTO_UPDATE` | `1` | Disable Homebrew auto-update |
| `HOMEBREW_NO_INSTALL_CLEANUP` | `1` | Disable Homebrew cleanup after installs |
