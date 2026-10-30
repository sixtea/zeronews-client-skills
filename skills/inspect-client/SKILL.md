---
name: zeronews-inspect-client
description: Inspect a local ZeroNews client through its runtime file, local API, service status, and logs. Use when asked whether the client is running, connected, configured, healthy, or currently serving endpoints.
license: MIT
metadata:
  author: zeronews
  version: "1.0"
compatibility: Works best when a local client runtime is running, but can still inspect workdir state and service state without an active runtime.
---

# Inspect Client

Inspect the local ZeroNews client without mutating state unless the user explicitly asks for a repair action.

`zeronews status --json` is **not** read-only when `{workdir}/login_pending.json` exists: it will poll the control plane and may write `config.yml`. For a read-only check, prefer `{workdir}/config.yml` plus the local API `/api/v1/status` when the runtime is healthy. If the user wants to finish a pending browser login, then `status --json` is the correct command.

Read these JSON fields; do not treat exit `3` as failure, and do not treat “JSON parsed” as logged in:

- `logged_in`
- `login_pending`
- `login_error`

## What this skill does

- reads `prg_runtime.json`
- checks the local health endpoint
- reads runtime status, config summary, and endpoint list from the local API
- falls back to service status and logs when the runtime file is stale or the process is not healthy

## Workflow

### Step 1: Find the runtime

Inspect:

- `{workdir}/prg_runtime.json`
- `{workdir}/config.yml`
- log files under `{workdir}/logs/`

Do not assume `prg_runtime.json` is current. Validate it with a health check.

### Step 2: Probe health

If a runtime file exists, probe:

- `GET http://{connect_addr}{health_path}`

If health succeeds, the runtime is live.

If health fails with a network or decode error, treat the runtime file as possibly stale and fall back to service and log inspection.

### Step 3: Query the local API

When the runtime is healthy, use:

- `GET /api/v1/status`
- `GET /api/v1/config`
- `GET /api/v1/endpoints`

Use the responses to answer:

- is the client configured
- is the client connected
- what client ID and cloud server address are active
- how many endpoints exist
- whether any endpoint is in an error state

### Step 4: Reconnect only when appropriate

`POST /api/v1/reconnect` is available, but do not trigger it automatically unless the user explicitly wants to restore the connection.

## Output

Return the most important runtime facts:

- configured or not
- connected or not
- client ID
- cloud server address
- endpoint count
- any endpoint errors
- whether the runtime file looked stale

## Safety rules

- prefer read-only inspection first
- do not trust `prg_runtime.json` without a health probe
- do not restart, reconnect, or reset unless the user asked for a repair action
