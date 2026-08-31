---
url: https://alchemy.run/cli/plan
title: "plan"
description: "Preview what would change without applying anything. Equivalent to alchemy deploy --dry-run."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

```sh
alchemy plan [options]
```

`plan` previews what would change without applying anything — it is equivalent to `alchemy deploy --dry-run`.

```
Plan: 1 to create, 1 to update

+ Queue (AWS.SQS.Queue)
~ Worker (Cloudflare.Worker)
```

The plan uses `+` for creates, `~` for updates, `-` for deletes, and `•` for no-ops. No approval prompt is shown and no changes are made.

## Detailed property changes

Pass `--detailed` to show declared resource inputs as YAML:

```sh
alchemy plan --detailed
```

```
Plan: 1 to create, 1 to update

+ EventsQueue
  properties:
    fifo: true
    visibilityTimeout: 30

~ OrderHandler
  before:
    memorySize: 512
    env:
[s6]MODE: development
  after:
    memorySize: 1024
    env:
[s6]MODE: production
```

Creates show their desired properties. Updates and replacements show the previously persisted declared properties followed by the desired properties; this is a declaration diff, not a live-cloud drift read. Outputs appear as `(known after apply)`, computed values as `(computed)`, and secrets remain redacted. Deletes stay compact.

## Flags

| Option | Description |
| --- | --- |
| `--config, -c <file>` | Stack file to plan (defaults to `alchemy.run.ts`) |
| `--stage <name>` | Stage to plan against (defaults to `dev_$USER`) |
| `--profile <name>` | Auth profile to use (defaults to `$ALCHEMY_PROFILE` or `default`) |
| `--env-file <path>` | Load environment variables from a file |
| `--detailed` | Show declared resource properties as YAML |

## Where next

- [deploy](deploy.md) — apply the plan
- [destroy](destroy.md) — delete everything in a stack
- [Inspecting State](inspecting-state.md) — examine what the plan diffs against
