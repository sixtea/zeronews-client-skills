---
name: zeronews-install-client
description: Download and install the ZeroNews client online on Linux, macOS, or Windows, or run it via Docker. Native installs bind the machine with zeronews login (browser confirm) unless the user already gave an AuthToken. Docker still requires TOKEN. Use when asked to install ZeroNews, set up the client from scratch, deploy via Docker, or get a new device online.
license: MIT
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires network access to download.zeronews.cc (native installs) or pull a ZeroNews Docker image. Linux/macOS/Windows native installs need a shell; Docker installs need Docker available on the host.
---

# Install Client

Install the ZeroNews client from official online sources, bind the machine (browser `zeronews login` by default, or AuthToken if the user provided one), start the runtime, and confirm the device appears online.

This skill covers the full first-time setup path. For auth-only changes on an already-installed client, prefer `zeronews-configure-client`. For service management after install, prefer `zeronews-manage-client-service`. For health checks, prefer `zeronews-inspect-client`.

## What this skill does

- detects or asks for the target platform: Linux, macOS, Windows, or Docker
- downloads the official client or pulls the official image
- binds the machine during install (native: `zeronews login`; Docker: `TOKEN` env)
- starts the client in foreground or background/service mode
- verifies installation through CLI output, local API, container logs, or console guidance

## Prerequisites

- native install: a browser the account owner can use to confirm (no AuthToken required)
- Docker install: AuthToken from the ZeroNews user console ("快速开始")
- outbound network access to ZeroNews download endpoints or Docker registries
- for native Linux service mode: elevated privileges when not already root
- for native macOS service mode: elevated privileges when not already root
- for native Windows service mode: administrator PowerShell or CMD
- for Docker: Docker installed and able to pull images

## Workflow

### Step 1: Pre-flight

Silently inspect when possible:

- current OS and CPU architecture (`uname -s`, `uname -m`, or Windows environment)
- whether `zeronews` / `zeronews.exe` is already installed
- whether Docker is available (`docker version`)
- whether an existing workdir or Docker data directory already contains `config.yml`

Ask all questions upfront before running any install command:

1. **Native vs Docker** — native Linux/macOS/Windows use `zeronews login` (browser confirm) unless the user already pasted an AuthToken. **Docker still requires AuthToken first.** Do not invent, guess, or reuse a token from unrelated context.
2. **Install method** — native Linux, native macOS, native Windows, or Docker.
3. **Start mode** — foreground (`start`) or background/service (`service install` + `service start`). Default to foreground unless the user asked for long-lived or production use.
4. **Optional client ID** — only if the user explicitly wants to bind a specific client ID.
5. **Docker-only choices** when Docker is selected:
   - image registry: China mirror (`docker.zeronews.cc/zeronews/zeronews`) or Docker Hub (`zeronews/zeronews`)
   - launch style: `docker run` or `docker compose`
   - host data directory, default `/opt/zeronews`
6. **macOS-only** — if architecture is unclear, ask whether the Mac is Apple Silicon or Intel, or detect with `uname -m`.
7. **Windows-only** — install directory, default a dedicated folder such as `D:\zeronews`.

If a client is already installed and configured, stop and ask whether to:

- skip install and only start or inspect
- re-auth with `zeronews-configure-client`
- reset and reinstall with `zeronews-reset-client` (destructive, explicit confirmation only)

### Step 2: Install by platform

Use the exact commands in `references/INSTALL-COMMANDS.md`. Do not substitute unofficial download URLs or install scripts.

Summary:

| Platform | Install | Bind |
| --- | --- | --- |
| Linux | official install script, token optional | default: `NO_TTY=1 zeronews login` then poll `zeronews status --json`; or `sh -s install {TOKEN}` |
| macOS | download binary to `/Applications/zeronews` | `NO_TTY=1 ./zeronews login` then poll `status --json` |
| Windows | download `zeronews.exe` | `zeronews.exe login` then poll `status --json` |
| Docker | `docker run` or `docker compose` | **must ask user for token first**, then inject `-e TOKEN=<TOKEN>` |

#### Docker auth rule

For Docker installs, **always ask the user for AuthToken before generating or running any command**. Never start a container with a placeholder token and promise to fix it later. Never write the token into a compose file on disk unless the user explicitly accepts that persistence risk.

Docker auth happens through the `TOKEN` environment variable. Do not run a separate `authtoken` command inside the container for first-time setup.

### Step 3: Bind the machine (native installs only)

If `zeronews status --json` already has `"login": true` (valid `config.yml`), skip this step. Do not ask for a token and do not open login again.

If the user pasted an AuthToken, keep the old path: Linux `sh -s install {TOKEN}`, or `zeronews authtoken {TOKEN}` on macOS/Windows.

Otherwise use browser login:

```bash
NO_TTY=1 zeronews --workdir {WORKDIR} login
```

Only add `--client {NAME}` when the host product name is known (for example Cursor). Do not hardcode `--client Cursor` when unknown.

Stdout first line is the verify URL. Open it locally. If the browser opened, do not paste the URL into the chat. If open failed, show the URL and say it is only for the account owner — do not forward it.

Then poll:

```bash
zeronews --workdir {WORKDIR} status --json
```

- exit `0` and `"login": true` → bound. If the service is not running, `service start`. If it is already running without the new config, `service restart` or `reload`.
- exit `3` and `"login_pending": true` → still waiting; poll again. This is not a failure. Do not treat JSON parse success as logged in.
- exit `1` and `"login_error"` → stop and report the error (`denied` / `expired` / `quota_exceeded` / `failed`)
- `NO_TTY` **exists** (including `NO_TTY=0`) means non-interactive: `login` prints the URL and exits; only `status --json` writes `config.yml`.

Linux sudo must use the same workdir as later commands (`/etc/zeronews` is the default for the official script).

Do not hand-edit `config.yml`. Do not call `zeronews authtoken` after a successful login poll.

### Step 3b: Handle `Local config file already exists, token update ignored`

If `authtoken` or the Linux install script leaves the client unauthenticated and the CLI prints:

```text
Local config file already exists, token update ignored
```

stop immediately. The new token was **not** applied. Do not continue to start or service-install steps.

Tell the user plainly what happened, then ask them to choose:

1. **Reset** — hand off to `zeronews-reset-client`, then re-run auth
2. **Ignore** — keep the existing config and stop the install/auth flow

Do not auto-reset. Do not manually delete files when the user chose reset. Follow `../configure-client/references/EXISTING-CONFIG-CONFLICT.md`.

If the user chooses **Reset**:

1. hand off to `zeronews-reset-client`
2. use elevated privileges when required; if `id -u` is already `0`, run without `sudo` (see `../configure-client/references/PRIVILEGE-EXECUTION.md`)
3. after reset succeeds, re-bind with `zeronews login` (or `authtoken {TOKEN}` / the Linux script with `{TOKEN}` if that was the original path)
4. continue install/start only after `login: true` or `Authentication successful`

If the user chooses **Ignore**:

- explain that the existing binding was kept
- skip start steps unless the user still wants to start the already-configured client
- hand off to `zeronews-inspect-client` if they want to verify current state

For Docker volumes that already contain `config.yml`, apply the same reset-or-ignore choice. If reset is chosen, hand off to `zeronews-reset-client` against the mounted host workdir before recreating the container with a new `TOKEN`.

### Step 4: Start the client

#### Native foreground

```bash
zeronews start
```

macOS:

```bash
cd /Applications/zeronews
./zeronews start
```

Windows:

```powershell
.\zeronews.exe start
```

#### Native background / service

Only when the user asked for long-lived operation.

Before running service commands, check whether the session is already root. If `id -u` is `0`, omit `sudo`:

```bash
# non-root session
sudo zeronews service install
sudo zeronews service start

# already root
zeronews service install
zeronews service start
```

Platform-specific privilege requirements are documented in `../configure-client/references/SERVICE-PRIVILEGES.md` and `../configure-client/references/PRIVILEGE-EXECUTION.md`.

#### Docker

Containers start detached as part of `docker run -d` or `docker compose up -d`. After launch, inspect with:

```bash
docker ps
docker logs zeronews --tail 50
```

See `references/DOCKER.md` for full run and compose examples.

### Step 5: Verify installation

Confirm success using the best available signal:

- native: command exit code, then `zeronews-inspect-client` flow if the local runtime is reachable
- Docker: container is running, recent logs show no fatal auth or startup error
- user console: device appears online

Return:

- install method used
- binary path, workdir, or container name
- config path (`config.yml` location or Docker volume mount)
- client ID and cloud server address when available
- start mode used
- verification result

## Output

Return a concise install report:

- platform and install method
- download URL or image name used
- config path
- client ID and cloud server address if resolved
- how the client was started
- whether the device appears online or what the user should check next

## Handoff

After install succeeds:

- add tunnels → `zeronews-expose-service`, `zeronews-expose-webhook`, or `zeronews-share-files`
- inspect runtime → `zeronews-inspect-client`
- manage an already-installed service → `zeronews-manage-client-service`

## Safety rules

- native installs default to `zeronews login`; Docker installs must still ask for AuthToken explicitly
- do not embed real tokens in examples, logs, or committed compose files
- do not use unofficial mirrors, scripts, or package sources
- do not claim UDP tunnel support
- do not install a system service unless the user asked for background/long-lived operation
- do not overwrite an existing configured client without explicit user approval
- when auth prints `Local config file already exists, token update ignored`, require an explicit reset-or-ignore choice before continuing
- when the user chooses reset, hand off to `zeronews-reset-client` instead of manually deleting files
- before any command that would use `sudo`, check whether the session is already root; if `id -u` is `0`, run without `sudo`
- prefer official commands from `references/INSTALL-COMMANDS.md` over improvised package managers
- on macOS, if Gatekeeper blocks the binary, tell the user to allow it in System Settings rather than disabling security globally
