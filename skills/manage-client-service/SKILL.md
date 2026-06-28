---
name: zeronews-manage-client-service
description: Manage the ZeroNews client as a native OS service with `zeronews service`. Use when asked to install, start, stop, restart, uninstall, or inspect the ZeroNews client service.
license: MIT
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires the `zeronews` CLI. Available behavior is platform-specific across Linux, macOS, and Windows.
---

# Manage Client Service

Use the built-in service management commands instead of trying to script your own daemon wrapper.

## What this skill does

- installs the client as a system service
- starts and stops the service
- restarts the service
- checks service status
- uninstalls the service when explicitly requested

## Workflow

### Step 1: Ask intent and detect privileges

Ask all questions upfront:

1. What action is needed: install, start, stop, restart, status, or uninstall?
2. Which workdir should the service use?

Silently inspect whether the session is already elevated:

- Linux / macOS: `id -u` → `0` means root
- Windows: Administrator shell check per `../configure-client/references/PRIVILEGE-EXECUTION.md`

If not elevated and the action requires it, tell the user to rerun with `sudo` or an Administrator shell. The agent cannot enter a password interactively.

### Step 2: Use the existing service commands

Examples for a non-root session:

```bash
sudo zeronews --workdir {WORKDIR} service install
sudo zeronews --workdir {WORKDIR} service start
zeronews --workdir {WORKDIR} service status
```

If `id -u` is already `0`, omit `sudo`:

```bash
zeronews --workdir {WORKDIR} service install
zeronews --workdir {WORKDIR} service start
zeronews --workdir {WORKDIR} service status
```

### Step 3: Respect OS differences

- Linux uses `systemd`
- macOS uses `launchd`
- Windows uses the Windows Service Manager

Do not paper over platform-specific privilege or file-path errors.

## Safety rules

- before any command that would use `sudo`, check whether the session is already root; if so, run without `sudo`
- do not install a service unless the user explicitly asked for service mode
- do not uninstall a service unless the user explicitly asked for removal
- do not claim service install succeeded without checking the command result
