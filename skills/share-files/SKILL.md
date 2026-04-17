---
name: zeronews-share-files
description: Share a local directory through the ZeroNews client using the supported `file_share` feature. Use when asked to publish a folder, browse files remotely, or expose a local directory through a ZeroNews tunnel.
license: UNLICENSED
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
2. public domain or server-assigned address

Build a supported `file_share` config and create the tunnel:

```bash
zeronews --workdir {WORKDIR} add https --feature=file_share --feature-config '{...}'
```

At minimum, the feature config must include:

```json
{
  "file_share": {
    "root_path": "/absolute/path"
  }
}
```

After the request is accepted, do not reload, reconnect, or restart the client just to make the file-sharing mapping appear. Wait for the endpoint to arrive normally.

## Safety rules

- do not use a relative `root_path`
- do not claim file-share support without `feature=file_share`
- do not ask the user for an internal local-port detail just to make file sharing work
- do not expose download, preview, or hidden-file options to users until those options are intentionally provided as part of the product
- if the user wants a TLS tunnel with file sharing, confirm the TLS setup first
- do not reload, reconnect, or restart the client after adding the file-sharing tunnel unless the user explicitly asked for a repair action
