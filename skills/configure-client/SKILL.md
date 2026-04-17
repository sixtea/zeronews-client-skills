---
name: zeronews-configure-client
description: Configure a ZeroNews client with `zeronews authtoken`, save local client config, and optionally prepare it for long-lived use. Use when asked to register, authenticate, bind, or initialize a ZeroNews client.
license: UNLICENSED
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

### Step 3: Handle existing config carefully

Do not overwrite an existing stopped-client config silently.

Expected behavior:

- if a running local client is reachable, the CLI can reconfigure the running client
- if no running local client is reachable and a config file already exists, the CLI may keep the existing config instead of replacing it

If replacement is required and the normal auth command refuses to replace the file, stop and ask the user whether to:

- start the client first and re-run auth
- or reset local state explicitly

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
- do not assume the runtime file means the local client is healthy; probe it first
