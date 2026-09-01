---
url: https://alchemy.run/cli/plan
title: "plan"
description: "Preview what would change without applying anything. Equivalent to alchemy deploy --dry-run."
access_date: 2026-09-01T03:40:51.295Z
current_date: 2026-09-01T03:40:51.295Z
---

```sh
alchemy plan [file] [options]
```

`plan` previews what would change without applying anything — it is equivalent to `alchemy deploy --dry-run`.

```
Plan: 1 to create, 1 to update

+ Queue (AWS.SQS.Queue)
~ Worker (Cloudflare.Worker)
```

The plan uses `+` for creates, `~` for updates, `-` for deletes, and `•` for no-ops. No approval prompt is shown and no changes are made.

## Flags

| Option | Description |
| --- | --- |
| `[file]` | Stack file to plan (defaults to `alchemy.run.ts`) |
| `--stage <name>` | Stage to plan against (defaults to `dev_$USER`) |
| `--profile <name>` | Auth profile to use (defaults to `default` or `$ALCHEMY_PROFILE`) |
| `--env-file <path>` | Load environment variables from a file |

## Where next

- [deploy](deploy.md) — apply the plan
- [destroy](destroy.md) — delete everything in a stack
- [Inspecting State](inspecting-state.md) — examine what the plan diffs against
