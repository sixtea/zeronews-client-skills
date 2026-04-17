---
name: zeronews-manage-client-service
description: Manage the ZeroNews client as a native OS service with `zeronews service`. Use when asked to install, start, stop, restart, uninstall, or inspect the ZeroNews client service.
license: UNLICENSED
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

### Step 1: Ask intent and privilege questions

Ask all questions upfront:

1. What action is needed: install, start, stop, restart, status, or uninstall?
2. Which workdir should the service use?
3. Does the user have the required elevated privileges on this machine?

### Step 2: Use the existing service commands

Examples:

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

- do not install a service unless the user explicitly asked for service mode
- do not uninstall a service unless the user explicitly asked for removal
- do not claim service install succeeded without checking the command result
