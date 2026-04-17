---
name: zeronews-expose-service
description: Expose a local service through the ZeroNews client by creating a tunnel with `zeronews add`. Use when asked to expose a local app, add a tunnel, publish a local service, or create a ZeroNews tunnel.
license: UNLICENSED
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires the `zeronews` CLI and an authenticated client config. Can use a running local client or operate directly through the normal CLI flow.
---

# Expose Service

Create a tunnel for a local service through the ZeroNews client.

## What this skill does

- creates a tunnel with `zeronews add`
- supports these tunnel types:
  - `https`
  - `tcp`
  - `tls`
- supports these optional features:
  - `webhook`
  - `file_share`

## Prerequisites

- `zeronews` command available
- client already configured with `config.yml`, or the user is willing to run the configure flow first

## Workflow

### Step 1: Pre-flight

Silently inspect:

- whether the client is configured
- whether a running local client is available
- whether the target local service is expected to be on a known port

Ask all questions upfront:

1. tunnel type: `https`, `tcp`, or `tls`
2. local port
3. optional local IP, or use the default behavior
4. optional public domain, or let the server allocate one
5. optional bandwidth limit in Mbps, or use the default
6. whether to enable a supported feature:
   - none
   - `webhook`
   - `file_share`

If the client is not configured, stop and use `zeronews-configure-client` first.

### Step 2: Create the tunnel

Prefer the CLI:

```bash
zeronews --workdir {WORKDIR} add {TYPE} --port={PORT}
```

Add flags only when the user actually asked for them:

- `--local_ip`
- `--domain`
- `--bw`
- `--mode`
- `--feature`
- `--feature-config`
- `--extra-config`

### Step 3: Feature handling

If the user wants `webhook` or `file_share`, build `--feature-config` using the supported examples in `references/FEATURE-CONFIGS.md`.

Do not invent extra feature names.

### Step 4: Wait for endpoint delivery

After the tunnel request is accepted, do not reload, reconnect, or restart the user's client just to make the mapping appear.

The correct behavior is:

- wait for the endpoint to arrive from the server
- if needed, observe endpoint state through the normal status and endpoint inspection flow

### Step 5: Batch mode when needed

If the user wants multiple tunnels at once, prefer:

- `--from-file`
- or `--from-stdin`

The CLI already supports reading YAML or JSON tunnel entries.

## Output

Return:

- tunnel ID
- protocol
- mode
- local target
- public URL if provided by the server

## Safety rules

- do not send multiple mutating client requests in parallel
- do not claim UDP is supported
- do not assume feature support beyond `webhook` and `file_share`
- do not reload, reconnect, or restart the client after adding a tunnel unless the user explicitly asked for a repair action
