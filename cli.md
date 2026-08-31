---
url: https://alchemy.run/cli
title: "CLI"
description: "Every alchemy command at a glance — the command map, common options, and how the interactive TUI decides when to render."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

```sh
bun alchemy <command> [options]
```

Every command operates on an `alchemy.run.ts` stack file (or a custom entrypoint passed with `--config <file>`, which must exist) and targets a [stage](environments/stages.md).

## Command map

```sh
alchemy
├─ deploy              # plan → approve → apply
├─ plan                # preview changes, apply nothing
├─ destroy             # delete every resource in a stage
├─ drift               # detect (and optionally repair) infrastructure drift
├─ unsafe nuke         # delete everything a provider can list, tracked or not
│
├─ dev                 # hot-reloading development loop
├─ logs                # fetch past log entries, or tail live with --tail
│
├─ profile create|rename|edit|refresh|current|list|show|delete
│                      # manage credentials and accounts (bare \`profile\` opens the dashboard)
│
├─ state list|read|delete        # inspect and manage the state store (bare \`state\` opens the explorer)
│
└─ provider
   ├─ check-env           # CI preflight: verify required provider env vars are set
   ├─ aws bootstrap|teardown
   └─ cloudflare bootstrap|teardown|token|state logs
```

Each command has its own page: [deploy](cli/deploy.md), [plan](cli/plan.md), [destroy](cli/destroy.md), [drift](cli/drift.md), [unsafe nuke](cli/nuke.md), [dev](cli/dev.md), [logs](cli/logs.md), [profile](cli/profile.md), [state](cli/state.md), [AWS provider commands](cli/aws.md), and [Cloudflare provider commands](cli/cloudflare.md). The workflow guides cover [adopting resources](cli/adopting-resources.md) and [inspecting state](cli/inspecting-state.md).

## Common options

| Option | Description |
| --- | --- |
| `--stage <name>` | Stage to target. Falls back to the `stage` env config, then `dev_$USER`. Must match `[a-z0-9]+([-_a-z0-9]+)*` (case-insensitive). |
| `--profile <name>` | Auth profile from `~/.alchemy/profiles.json`. Overrides `$ALCHEMY_PROFILE`; defaults to `default`. |
| `--env-file <path>` | File to load environment variables from. Defaults to `.env`. |
| `--yes` | Yes to all prompts. |
| `--no-input` | Disable prompts and the interactive TUI. Commands that would need input fail instead of hanging. |
| `--config, -c <file>` | Stack entrypoint. Defaults to `alchemy.run.ts`; must exist. |

Use `--profile` with `deploy`, `plan`, `destroy`, `dev`, `logs`, `drift`, or `state`.

## The interactive TUI

In an interactive terminal, `deploy`, `destroy`, and `plan` render a live TUI: the rendered plan, an arrow-key approval prompt, and per-resource apply progress that unmounts into the final output. Everywhere else the same commands emit plain line-oriented logs.

The `--no-input` flag and environment variables override the detection, in this order:

| Signal | Effect |
| --- | --- |
| `--no-input` flag | Force plain output |
| `ALCHEMY_PLAIN=1` or `ALCHEMY_NO_TUI=1` | Force plain output |
| `ALCHEMY_TUI=1` | Force the TUI |
| No TTY, `CI=1`, or a known agent env var (`CLAUDECODE`, `CLAUDE_CODE_ENTRYPOINT`, `CURSOR_AGENT`, `AIDER_MODEL`, `CODEX_CLI`) | Force plain output |

Interactive terminals use the animated TUI; redirected and non-interactive runs use append-only output.

Plain mode never prompts — it prints the plan, makes no changes, and fails with `error: Cannot approve this operation without terminal input. Pass --yes to continue.` In CI, always pass `--yes`.

## Exit codes

Exit codes are script-safe: `0` only means the command actually completed.

| Code | Meaning |
| --- | --- |
| `0` | Command completed |
| `1` | Failure, including a declined plan or bare `profile` / `state` without a terminal |
| `130` | Cancelled by the user (Ctrl-C, or Escape inside a prompt) |

Running these commands in a pipeline? See [CI](environments/ci.md).

## Where next

- [deploy](cli/deploy.md) — plan, approve, and apply changes
- [dev](cli/dev.md) — hot-reloading development loop
- [profile](cli/profile.md) manages credentials and connected accounts
- [state](cli/state.md) — inspect and manage the state store
- [Stages](environments/stages.md) — how environments are isolated
- [CI](environments/ci.md) — run the CLI in pipelines
