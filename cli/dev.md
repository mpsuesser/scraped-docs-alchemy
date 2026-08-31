---
url: https://alchemy.run/cli/dev
title: "dev"
description: "Run your stack in development mode with hot reloading — resources deploy to the cloud while Workers run in the local dev runtime."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

```sh
alchemy dev [options]
```

Run your stack in development mode with hot reloading. Resources are deployed to the cloud while Workers run in the local dev runtime; file changes trigger automatic rebuilds and hot reloads.

## How it works

`dev` re-runs your stack under `bun --watch` (or `node --watch` with type-stripping flags on supported Node versions) via `alchemy/bin/exec`.

`--yes` and dev mode are implied, so approval prompts never appear.

A failed apply keeps dev alive so healthy resources keep serving — fix the error and save to redeploy.

## Flags

| Option              | Description                                                        |
| ------------------- | ------------------------------------------------------------------ |
| `--stage <name>`    | Stage to use for dev (defaults to `dev_$USER`)                     |
| `--force`           | Force updates for resources that would otherwise no-op             |
| `--profile <name>`  | Auth profile to use (defaults to `$ALCHEMY_PROFILE` or `default`) |
| `--env-file <path>` | Load environment variables from a file                             |
| `--config, -c <file>`            | Stack file to run (defaults to `alchemy.run.ts`)                   |

## Examples

```sh
# Start dev mode
alchemy dev

# Use a custom stage
alchemy dev --stage dev
```

## Where next

- [logs](logs.md) fetches past logs or tails them with `--tail`
- [deploy](deploy.md) — deploy for real
- [Local development](../environments/local-development.md) — how dev mode works under the hood
