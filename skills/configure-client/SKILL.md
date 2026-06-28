---
name: zeronews-configure-client
description: Configure a ZeroNews client with `zeronews authtoken`, save local client config, and optionally prepare it for long-lived use. Use when asked to register, authenticate, bind, or initialize a ZeroNews client.
license: MIT
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires the `zeronews` CLI. The skill may work against a stopped client or a running local client runtime.
---

# Configure Client

Configure a ZeroNews client by using the existing auth command instead of editing local files manually.

## What this skill does

- binds an auth token to the client
- writes the local `config.yml`
- returns the resolved client ID and cloud server address
- can work through the running local client when it is already active

## Prerequisites

- `zeronews` command available
- auth token issued by the control-plane

## Workflow

### Step 1: Pre-flight

Silently inspect:

- whether `zeronews` is installed
- the intended workdir
- whether `{workdir}/config.yml` already exists
- whether `{workdir}/prg_runtime.json` points to a healthy running local client

Ask all questions upfront:

1. What auth token should be used?
2. Should a specific `client_id` be supplied, or should the server decide?
3. Use the default workdir, or a specific absolute `--workdir` path?
4. Is this only a local one-time setup, or should the client also be prepared for service mode afterward?

If `config.yml` already exists, ask whether the user wants to keep the current binding or replace it.

### Step 2: Authenticate

Prefer the CLI:

```bash
zeronews --workdir {WORKDIR} authtoken {TOKEN}
```

If the user provides a client ID:

```bash
zeronews --workdir {WORKDIR} authtoken {TOKEN} {CLIENT_ID}
```

### Step 3: Handle `Local config file already exists, token update ignored`

If `authtoken` prints:

```text
Local config file already exists, token update ignored
```

stop immediately. The new token was **not** applied. Do not continue as if auth succeeded.

Present a clear choice to the user:

1. **Reset** — hand off to `zeronews-reset-client`, then re-run `authtoken`
2. **Ignore** — keep the existing config and stop the auth attempt

Do not auto-reset. Do not manually delete files when the user chose reset. Follow `references/EXISTING-CONFIG-CONFLICT.md`.

Expected behavior before the warning appears:

- if a running local client is reachable, the CLI can reconfigure through the local API without this warning
- if no running local client is reachable and `config.yml` already exists, the CLI keeps the old config and prints the warning

If the user chooses **Reset**:

1. hand off to `zeronews-reset-client`
2. use elevated privileges when required; if `id -u` is already `0`, run without `sudo` (see `references/PRIVILEGE-EXECUTION.md`)
3. after reset succeeds, re-run `zeronews --workdir {WORKDIR} authtoken {TOKEN}`
4. continue only after the CLI prints `Authentication successful`

If the user chooses **Ignore**:

- explain that the existing binding was kept
- do not retry auth unless the user changes their mind
- hand off to `zeronews-inspect-client` if they want to verify current state

### Step 4: If the user asked for long-lived operation

Do not install or start a system service unless the user explicitly asked for it.

If they did, hand off to the service management flow after auth succeeds.

## Output

Return:

- config path
- client ID
- cloud server address

## Safety rules

- do not write `config.yml` by hand when `zeronews authtoken` can do it
- do not delete an existing config without explicit user approval
- when auth prints `Local config file already exists, token update ignored`, require an explicit reset-or-ignore choice; never auto-delete config
- when the user chooses reset, hand off to `zeronews-reset-client` instead of manually deleting files
- before any command that would use `sudo`, check whether the session is already root; if `id -u` is `0`, run without `sudo`
- do not assume the runtime file means the local client is healthy; probe it first
