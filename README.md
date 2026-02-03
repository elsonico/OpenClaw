---
summary: "Runbook: Docker gateway on Ubuntu 22.04 with host volumes and macOS export"
read_when:
  - You want OpenClaw in Docker on Ubuntu 22.04
  - You want to export an existing macOS gateway into host volumes
---

# Ubuntu 22.04 Docker runbook

This runbook sets up a Dockerized OpenClaw Gateway on **Ubuntu 22.04** with
host-mounted volumes so you can copy your macOS state and workspace into the
Linux host and keep it visible outside the container.

## What you get

- OpenClaw Gateway running in Docker
- Host volumes at `/home/<user>/OpenClaw/config` and `/home/<user>/OpenClaw/workspace`
- A migration path from macOS without redoing onboarding

## Prerequisites

- Ubuntu 22.04 host
- Docker Engine + Docker Compose v2 already installed
- `git` and `rsync` available (install via Ubuntu packages if missing)

## Safety assumptions (existing containers running)

This runbook does **not** touch or modify existing containers.

- We **do not** stop, remove, prune, or upgrade anything.
- We use a **new image tag** and a **separate compose project**.
- We avoid port collisions by confirming `18789/18790` are free (or override them).

## Step 1 - Verify Docker is working (no changes)

```bash
docker --version
docker ps
```

If Docker is installed and your existing containers are running, continue.

## Step 2 - Prepare host volumes on Ubuntu

```bash
mkdir -p "/home/$USER/OpenClaw/config" "/home/$USER/OpenClaw/workspace"
```

## Step 3 - Clone OpenClaw on Ubuntu

```bash
mkdir -p ~/git
git clone https://github.com/openclaw/openclaw.git ~/git/openclaw
cd ~/git/openclaw
```

The `Dockerfile` lives in the repo root. If you ran `docker build` elsewhere,
`cd ~/git/openclaw` first.

## Step 4 - Export state from macOS

On macOS, stop the Gateway **from the OpenClaw Mac app** so the files are not
changing mid-copy.

Find your state directory (default: `~/.openclaw/`) and workspace
(`~/.openclaw/workspace/`). If you use a profile or custom state dir, copy that
instead. See [Migrating](/install/migrating).

Create tarballs:

```bash
cd ~
tar -czf openclaw-state.tgz .openclaw
tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Copy them to Ubuntu:

```bash
rsync -avz openclaw-state.tgz openclaw-workspace.tgz <user>@<gateway-host>:/home/<user>/OpenClaw/
```

On Ubuntu, extract into the host volumes:

```bash
cd "/home/$USER/OpenClaw"
tar -xzf openclaw-state.tgz -C "/home/$USER/OpenClaw/config" --strip-components=1
tar -xzf openclaw-workspace.tgz -C "/home/$USER/OpenClaw/workspace" --strip-components=2
```

If you used a profile or custom state dir, adjust paths to match your layout.

## Step 5 - Choose safe ports (avoid collisions)

Check whether the default ports are free:

```bash
sudo ss -ltnp | rg ":18789|:18790"
```

If you see any listeners, pick open ports and set these env vars in the next step.

## Step 6 - Configure Docker environment

From the repo root on Ubuntu:

```bash
export OPENCLAW_CONFIG_DIR="/home/$USER/OpenClaw/config"
export OPENCLAW_WORKSPACE_DIR="/home/$USER/OpenClaw/workspace"
export OPENCLAW_GATEWAY_TOKEN="<your-token>"
export OPENCLAW_GATEWAY_PORT="18789"
export OPENCLAW_BRIDGE_PORT="18790"
export OPENCLAW_IMAGE="openclaw:local"
```

If you already have a token in the exported state, reuse it. Otherwise set a new
one and update your client later.

## Step 7 - Build and start the Gateway container (isolated project)

From the repo root:

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose -p openclaw up -d openclaw-gateway
```

`-p openclaw` creates a dedicated project name so you do not interfere with
other compose stacks on the host.

## Step 8 - Verify

```bash
docker compose -p openclaw exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

Open the UI:

- Local host: `http://127.0.0.1:<OPENCLAW_GATEWAY_PORT>/`
- Remote access: use SSH tunneling or expose the port explicitly

## Notes

- Treat `/home/<user>/OpenClaw/config` as secrets. It includes credentials and channel
  state.
- If ownership is wrong, fix it on the host: `sudo chown -R "$USER:$USER" "/home/$USER/OpenClaw"`
- For a fresh Docker setup (no migration), use [Docker](/install/docker) and run
  `./docker-setup.sh`

## Related

- [Docker](/install/docker)
- [Migrating](/install/migrating)
