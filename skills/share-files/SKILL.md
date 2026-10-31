---
name: zeronews-share-files
description: Share a local directory through the ZeroNews client using the supported `file_share` feature. Use when asked to publish a folder, browse files remotely, or expose a local directory through a ZeroNews tunnel.
license: MIT
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires an authenticated client and the `zeronews` CLI. Uses the existing tunnel creation flow with a `file_share` feature config.
---

# Share Files

This is a feature-focused wrapper around `zeronews add ... --feature=file_share`.

## Workflow

Ask all questions upfront:

1. which local directory should be shared
2. admin password for the share (at least 8 characters; required when creating)
3. public domain or server-assigned address

Build a supported `file_share` config and create the tunnel:

```bash
zeronews --workdir {WORKDIR} add https --feature=file_share --feature-config '{...}'
```

At minimum, the feature config must include:

```json
{
  "file_share": {
    "root_path": "/absolute/path",
    "admin_password": "<at least 8 characters>"
  }
}
```

Creating a share without `admin_password` is not a supported config. The control plane requires it (minimum 8 characters).

If an admin password is set but the client has not been upgraded, the public URL may still serve the whole share as anonymous read-only (list and download). Do not tell the user that filling in a password will make an old client require login.

After the request is accepted, do not reload, reconnect, or restart the client just to make the file-sharing mapping appear. Wait for the endpoint to arrive normally.

## Safety rules

- do not use a relative `root_path`
- do not create a file_share tunnel without `admin_password` of at least 8 characters
- do not omit `admin_password` from the create example or treat `root_path` as sufficient by itself
- do not use an empty `admin_password` to turn off NAS / file-share login
- do not claim that an old client will require login just because a password was set
- do not claim file-share support without `feature=file_share`
- do not ask the user for an internal local-port detail just to make file sharing work
- do not expose download, preview, or hidden-file options to users until those options are intentionally provided as part of the product
- if the user wants a TLS tunnel with file sharing, confirm the TLS setup first
- do not reload, reconnect, or restart the client after adding the file-sharing tunnel unless the user explicitly asked for a repair action
