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

### Local (Docker Compose)

1. Copy the example env file and fill in your API keys:

   ```bash
   cp .env.example .env
   # edit .env with your favourite editor
   ```

2. Start the container:

   ```bash
   docker compose up
   ```

The workspace directory is persisted in a Docker volume (`openclaw-workspace`) mounted at `/home/node/.openclaw/workspace` inside the container.

## Deploying on a VPS with Dokploy

[Dokploy](https://dokploy.com) is a free, self-hosted PaaS that runs on any VPS and lets you deploy Docker Compose projects through a web UI.

### Prerequisites

- A VPS with at least **2 GB RAM** and **40 GB disk** running Ubuntu 22.04 / Debian 12 (or compatible). The Docker image includes Chromium, Homebrew and all dependencies and can exceed 10 GB; allocating more disk ensures room for logs and workspace data.
- A domain or sub-domain pointed at your VPS IP (optional but recommended).

### 1 – Install Dokploy on the VPS

```bash
curl -sSL https://dokploy.com/install.sh | sh
```

Once the installer finishes it will print the Dokploy dashboard URL (usually `http://<your-vps-ip>:3000`). Open it in your browser and create your admin account.

### 2 – Create a new project

1. Click **Projects → New Project** in the Dokploy dashboard.
2. Give it a name (e.g. `openclaw`).

### 3 – Add a Docker Compose service

1. Inside the project click **Create Service → Docker Compose**.
2. Under **Source** choose **Git** and paste the repository URL:
   ```
   https://github.com/holk26/Openclaw-docker
   ```
3. Set the **Branch** to `main` (or the branch you want to deploy).
4. Leave the **Compose file** field as `docker-compose.yml`.

### 4 – Configure environment variables

In the **Environment** tab of the service, paste the contents of [`.env.example`](.env.example) and fill in the required values:

```env
NODE_ENV=production
HOMEBREW_NO_AUTO_UPDATE=1
HOMEBREW_NO_INSTALL_CLEANUP=1

# Gateway
OPENCLAW_GATEWAY_TOKEN=your-secret-token
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_BRIDGE_PORT=18790
OPENCLAW_GATEWAY_BIND=lan
GATEWAY_CONTROLUI_ALLOWEDORIGINS=https://agente.example.com

# Auth
AUTH_USERNAME=admin
AUTH_PASSWORD=your-strong-password

# Plugins  (comma-separated)
OPENCLAW_PLUGINS=discord,memory-core,whatsapp

# Integrations
WHATSAPP_ENABLED=true
TELEGRAM_BOT_TOKEN=

# AI provider  (at least one required)
OPENAI_API_KEY=
OPENROUTER_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_GENERATIVE_AI_API_KEY=
KIMI_API_KEY=

# Email
RESEND_API_KEY=
K4X_EMAIL_PASSWORD=

# Extra apt packages to install at startup (space-separated)
OPENCLAW_DOCKER_APT_PACKAGES=git curl nano
```

> **Tip:** You can also upload a `.env` file directly in the *Environment* tab.

### 5 – Deploy

Click **Deploy**. Dokploy will pull the repository, build the Docker image, and start the container.  
Logs are available in real time under the **Logs** tab.

### 6 – Access the service

If you exposed a port in `docker-compose.yml` you can reach it via `http://<your-vps-ip>:<port>`.  
To add HTTPS, go to **Domains** in the Dokploy dashboard and attach a domain – Dokploy will provision a Let's Encrypt certificate automatically.

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

All variables are documented in [`.env.example`](.env.example). Copy that file to `.env` before running.

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `production` | Node.js environment |
| `HOMEBREW_NO_AUTO_UPDATE` | `1` | Disable Homebrew auto-update |
| `HOMEBREW_NO_INSTALL_CLEANUP` | `1` | Disable Homebrew cleanup after installs |
| `OPENCLAW_GATEWAY_TOKEN` | _(required)_ | Secret token for gateway authentication |
| `OPENCLAW_GATEWAY_PORT` | `18789` | Port the gateway listens on |
| `OPENCLAW_BRIDGE_PORT` | `18790` | Port the bridge service listens on |
| `OPENCLAW_GATEWAY_BIND` | `lan` | Network interface to bind (`lan` or `all`) |
| `GATEWAY_CONTROLUI_ALLOWEDORIGINS` | _(empty)_ | Allowed origins for the control UI (comma-separated URLs) |
| `AUTH_USERNAME` | _(required)_ | Basic-auth username for the web interface |
| `AUTH_PASSWORD` | _(required)_ | Basic-auth password for the web interface |
| `OPENCLAW_PLUGINS` | _(empty)_ | Comma-separated list of plugins to load (e.g. `discord,memory-core,whatsapp`) |
| `WHATSAPP_ENABLED` | `false` | Enable the WhatsApp integration |
| `TELEGRAM_BOT_TOKEN` | _(empty)_ | Telegram bot token (from @BotFather) |
| `OPENAI_API_KEY` | _(empty)_ | OpenAI API key |
| `ANTHROPIC_API_KEY` | _(empty)_ | Anthropic / Claude API key |
| `GOOGLE_GENERATIVE_AI_API_KEY` | _(empty)_ | Google Gemini API key |
| `OPENROUTER_API_KEY` | _(empty)_ | OpenRouter API key (access to many models) |
| `KIMI_API_KEY` | _(empty)_ | Kimi (Moonshot AI) API key |
| `RESEND_API_KEY` | _(empty)_ | Resend transactional email API key |
| `K4X_EMAIL_PASSWORD` | _(empty)_ | K4X email account password |
| `OPENCLAW_DOCKER_APT_PACKAGES` | _(empty)_ | Extra apt packages to install at startup (space-separated) |
