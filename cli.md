---
url: https://alchemy.run/cli
title: "CLI"
description: "Every alchemy command at a glance — the command map, common options, and how the interactive TUI decides when to render."
access_date: 2026-08-06T07:23:05.654Z
current_date: 2026-08-06T07:23:05.654Z
---

```sh
bun alchemy <command> [file] [options]
```

Every command operates on an `alchemy.run.ts` stack file (or a custom entrypoint passed as the positional `[file]`, which must exist) and targets a [stage](environments/stages.md).

## Command map

```sh
alchemy
├─ deploy [file]        # plan → approve → apply
├─ plan [file]          # preview changes, apply nothing
├─ destroy [file]       # delete every resource in a stage
├─ unsafe nuke [file]   # delete everything a provider can list, tracked or not
│
├─ dev [file]           # hot-reloading development loop
├─ tail [file]          # stream live logs from deployed resources
├─ logs [file]          # fetch past log entries
│
├─ login [file]         # authenticate every provider in your stack
├─ profile show|clear   # inspect or wipe stored credentials
│
├─ state stacks|stages|resources|get|export|tree|clear   # inspect and manage the state store
│
├─ aws bootstrap        # set up the per-account AWS assets bucket
└─ cloudflare bootstrap|create-token|state logs   # state-store worker, API tokens, its logs
```

Each command has its own page: [deploy](cli/deploy.md), [plan](cli/plan.md), [destroy](cli/destroy.md), [unsafe nuke](cli/nuke.md), [dev](cli/dev.md), [tail](cli/tail.md), [logs](cli/logs.md), [login](cli/login.md), [profile](cli/profile.md), [state](cli/state.md), [aws](cli/aws.md), [cloudflare](cli/cloudflare.md) — plus two workflow guides, [Adopting Resources](cli/adopting-resources.md) and [Inspecting State](cli/inspecting-state.md).

## Common options

| Option | Description |
| --- | --- |
| `--stage <name>` | Stage to target. Falls back to the `stage` env config, then `dev_$USER`. Must match `[a-z0-9]+([-_a-z0-9]+)*` (case-insensitive). |
| `--profile <name>` | Auth profile from `~/.alchemy/profiles.json`. Defaults to `default` or `$ALCHEMY_PROFILE`. |
| `--env-file <path>` | File to load environment variables from. Defaults to `.env`. |
| `--yes` | Yes to all prompts. |
| `[file]` | Stack entrypoint. Defaults to `alchemy.run.ts`; must exist. |

`--profile` is genuinely common — `deploy`, `plan`, `destroy`, `dev`, `tail`, `logs`, `login`, and `state` all take it.

## The interactive TUI

In an interactive terminal, `deploy`, `destroy`, and `plan` render a live Ink TUI: the rendered plan, an arrow-key approval prompt, and per-resource apply progress that unmounts into the final output. Everywhere else the same commands emit plain line-oriented logs.

Environment variables override the detection, in this order:

| Variable | Effect |
| --- | --- |
| `ALCHEMY_PLAIN=1` or `ALCHEMY_NO_TUI=1` | Force plain output |
| `ALCHEMY_TUI=1` | Force the TUI |
| No TTY, `CI=1`, or a known agent env var (`CLAUDECODE`, `CLAUDE_CODE_ENTRYPOINT`, `CURSOR_AGENT`, `AIDER_MODEL`, `CODEX_CLI`) | Force plain output |

The TUI module is lazily imported only when selected, so plain runs never pay the Ink import cost.

Plain mode never prompts — it prints the plan, makes no changes, and reports: `Non-interactive terminal detected. Pass --yes to approve, or set ALCHEMY_TUI=1 for the interactive UI.` In CI, always pass `--yes`.

Running these commands in a pipeline? See [CI](environments/ci.md).

## Where next

- [deploy](cli/deploy.md) — plan, approve, and apply changes
- [dev](cli/dev.md) — hot-reloading development loop
- [login](cli/login.md) — authenticate your stack’s providers
- [state](cli/state.md) — inspect and manage the state store
- [Stages](environments/stages.md) — how environments are isolated
- [CI](environments/ci.md) — run the CLI in pipelines
