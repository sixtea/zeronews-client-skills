---
name: zeronews-reset-client
description: Reset local ZeroNews client state with `zeronews reset`. Use only when the user explicitly wants to remove service registration and local client state.
license: UNLICENSED
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires the `zeronews` CLI. This is destructive.
---

# Reset Client

Reset the local client only on explicit user request.

## What this skill does

The reset flow:

- attempts to uninstall the system service
- removes `config.yml`
- removes `control-plane.request.lock`
- removes `prg_runtime.json`
- removes the log directory
- removes the workdir if it becomes empty

## Workflow

### Step 1: Confirm intent clearly

Ask the user to confirm they want local client state removed.

Do not use this as a convenience shortcut for routine auth changes unless the user agreed.

### Step 2: Run the built-in reset

```bash
zeronews --workdir {WORKDIR} reset
```

### Step 3: Explain what was removed

Report the removed paths and whether service uninstall was skipped, completed, or blocked by privileges.

## Safety rules

- this skill is destructive
- do not run it to work around a smaller issue if inspection or re-auth is enough
- surface privilege failures plainly instead of pretending reset completed
