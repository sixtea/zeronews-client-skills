# ZeroNews Client Skills

A collection of Agent Skills for AI coding agents working with the ZeroNews client.

These skills are written for real user tasks:

- install the client online on Linux, macOS, Windows, or Docker
- configure a client
- expose a local service
- inspect runtime status
- manage the client service
- create supported webhook tunnels
- share a local folder
- reset local client state when explicitly requested

## Available Skills

### zeronews-install-client

Download and install the ZeroNews client from official online sources on Linux, macOS, or Windows, or run it through Docker. Binds AuthToken during setup and starts the client.

### zeronews-configure-client

Configure a ZeroNews client with `zeronews authtoken`, save local config, and optionally prepare it for service mode.

### zeronews-expose-service

Expose a local service with `zeronews add`. This skill focuses on the supported tunnel types documented here: `https`, `tcp`, and `tls`.

### zeronews-inspect-client

Inspect local client status through the runtime file, the local API, service status, and logs.

### zeronews-manage-client-service

Install, start, stop, restart, uninstall, and inspect the ZeroNews client as a native operating-system service.

### zeronews-expose-webhook

Create a supported webhook tunnel:

- standard webhook proxy
- verified WeChat Pay webhook
- verified WeChat Contract webhook

### zeronews-share-files

Create a file-sharing tunnel with a supported `file_share` feature config. Creating a share requires `root_path` and `admin_password` (at least 8 characters). If a password is set but the client is not upgraded, the public URL may still serve the whole share as anonymous read-only.

### zeronews-reset-client

Reset local client state with `zeronews reset`. This is destructive and should only be used when the user explicitly asks for it.

## Design Rules

- Prefer the CLI for mutating actions:
  - `zeronews authtoken`
  - `zeronews add`
  - `zeronews service ...`
  - `zeronews reset`
- Prefer the local API for runtime inspection:
  - `/healthz`
  - `/api/v1/status`
  - `/api/v1/config`
  - `/api/v1/endpoints`
  - `/api/v1/reconnect`
- Do not edit `config.yml` by hand when the existing CLI can persist it.
- Before any command that would use `sudo`, check whether the session is already root (`id -u` is `0` on Linux/macOS). If so, run the command directly without `sudo`.
- Do not assume UDP tunnel support.
- Do not invent unsupported features. The supported feature families documented here are:
  - `webhook`
  - `file_share`

## Examples

```text
Install ZeroNews on this Linux server with my auth token and start it as a service
```

```text
Set up ZeroNews in Docker and use my auth token
```

```text
Download and install the ZeroNews client on my Mac, then start it
```

```text
Configure this client with my auth token and make it run as a service
```

```text
Expose my app on port 3000 through ZeroNews
```

```text
Show me whether the local client is connected and what endpoints it has
```

```text
Create a WeChat Pay webhook tunnel for my local callback server
```

```text
Share a local folder through the client
```

## Repository Structure

- `skills/<name>/SKILL.md` - the skill definition and workflow
- `skills/<name>/references/` - supporting details when they improve safety or usability

This repository is intentionally user-facing. It explains how to use the client safely and consistently, without exposing internal implementation details.

## License

MIT
