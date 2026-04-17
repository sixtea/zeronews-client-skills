---
name: zeronews-expose-webhook
description: Create a ZeroNews webhook tunnel using the supported `webhook` configuration shapes. Use when asked to expose a webhook callback, verify WeChat Pay callbacks, or verify WeChat Contract callbacks.
license: UNLICENSED
metadata:
  author: zeronews
  version: "1.0"
compatibility: Requires an authenticated client and the `zeronews` CLI. Uses the same tunnel creation flow as `zeronews-expose-service`.
---

# Expose Webhook

This is a feature-focused wrapper around `zeronews add ... --feature=webhook`.

## Supported webhook modes

- standard reverse-proxy webhook
- verified WeChat Pay webhook
- verified WeChat Contract webhook

## Workflow

Ask all questions upfront:

1. local port
2. public domain or server-assigned address
3. standard webhook proxy, verified WeChat Pay, or verified WeChat Contract
4. inline key-value config, or config loaded from a YAML file

Then create the tunnel with:

```bash
zeronews --workdir {WORKDIR} add https --port={PORT} --feature=webhook --feature-config '{...}'
```

Use the supported JSON shapes from `../expose-service/references/FEATURE-CONFIGS.md`.

After the request is accepted, do not reload, reconnect, or restart the client just to make the webhook mapping appear. Wait for the endpoint to arrive normally.

## Safety rules

- do not invent webhook verify scenes other than `wechat_pay` and `wechat_contract`
- do not claim verification is active unless the feature config shape matches the supported examples
- do not reload, reconnect, or restart the client after adding the webhook tunnel unless the user explicitly asked for a repair action
