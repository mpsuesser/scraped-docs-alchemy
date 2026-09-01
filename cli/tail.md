---
url: https://alchemy.run/cli/tail
title: "tail"
description: "Stream live logs from deployed resources in real time, interleaved and color-coded by resource."
access_date: 2026-09-01T03:40:51.295Z
current_date: 2026-09-01T03:40:51.295Z
---

```sh
alchemy tail [file] [options]
```

Stream live logs from deployed resources in real time.

```
Tailing: Worker, Api

2026-04-15 14:32:01.123 PST [Worker] GET /hello.txt 200
2026-04-15 14:32:01.456 PST [Worker] PUT /world.txt 201
2026-04-15 14:32:02.789 PST [Api] POST /api/data 200
```

Logs from multiple resources are interleaved and color-coded by resource. The command streams indefinitely until you interrupt it with `Ctrl+C`.

`--filter` values are validated against the stack’s logical IDs — an unknown ID fails the command and lists the available IDs.

Only deployed resources whose provider implements tailing are streamed; if none qualify, the command prints `No tailable resources found. Deploy first, then run tail.` and exits.

## Flags

| Option | Description |
| --- | --- |
| `--stage <name>` | Stage to tail (defaults to `dev_$USER`) |
| `--filter <ids>` | Comma-separated logical resource IDs to include (e.g. `Worker,Api`) |
| `--profile <name>` | Auth profile to use (defaults to `default` or `$ALCHEMY_PROFILE`) |
| `--env-file <path>` | Load environment variables from a file |
| `[file]` | Stack file to tail (defaults to `alchemy.run.ts`) |

## Examples

```sh
# Tail only the Worker resource
alchemy tail --filter Worker

# Tail a specific stage
alchemy tail --stage prod
```

## Where next

- [logs](logs.md) — fetch a batch of past log entries instead of streaming
- [dev](dev.md) — run your stack with hot reloading
