---
url: https://alchemy.run/cli/destroy
title: "destroy"
description: "Delete every resource in a stack — plan all existing resources for deletion, ask for approval, and remove them in dependency order."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

```sh
alchemy destroy [file] [options]
```

`destroy` deletes every resource in a stack. It computes a plan where all existing resources are marked for deletion, asks for approval, and removes them in dependency order.

Under the hood, `destroy` is `deploy` with the desired state zeroed out — every resource in state plans as a deletion.

```
Plan: 2 to delete

- Worker (Cloudflare.Worker)
- Bucket (Cloudflare.R2.Bucket)

Proceed?
◉ Yes ○ No
✗ Worker (Cloudflare.Worker) deleted
✗ Bucket (Cloudflare.R2.Bucket) deleted
```

## Flags

| Option | Description |
| --- | --- |
| `[file]` | Stack file to destroy (defaults to `alchemy.run.ts`) |
| `--stage <name>` | Stage to destroy (defaults to `dev_$USER`) |
| `--yes` | Skip the approval prompt |
| `--dry-run` | Show what would be deleted without actually deleting |
| `--profile <name>` | Auth profile to use (defaults to `default` or `$ALCHEMY_PROFILE`) |
| `--env-file <path>` | Load environment variables from a file |

## Examples

```sh
# Destroy a PR preview environment
alchemy destroy --stage pr-42 --yes

# Destroy your personal dev stage
alchemy destroy --stage dev_sam
```

`destroy` is scoped to one stack + stage and driven by the state store; to enumerate and delete everything a set of providers can see in the live account, see [nuke](nuke.md).

## Where next

- [deploy](deploy.md) — bring the stack back
- [nuke](nuke.md) — account-wide teardown, not scoped to a stack
- [state](state.md) — inspect and clear the state the plan is driven by
- [Stages](../environments/stages.md) — how environments are isolated
