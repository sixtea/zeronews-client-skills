# ZeroNews Client Skills

A collection of Agent Skills for AI coding agents working with the ZeroNews client.

These skills are written for real user tasks:

- configure a client
- expose a local service
- inspect runtime status
- manage the client service
- create supported webhook tunnels
- share a local folder
- reset local client state when explicitly requested

## Available Skills

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

Create a file-sharing tunnel with a supported `file_share` feature config.

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
- Do not assume UDP tunnel support.
- Do not invent unsupported features. The supported feature families documented here are:
  - `webhook`
  - `file_share`

## Examples

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
