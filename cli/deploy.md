---
url: https://alchemy.run/cli/deploy
title: "deploy"
description: "Compute a plan, ask for approval, and create/update/delete resources to match the desired state."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

```sh
alchemy deploy [file] [options]
```

`deploy` computes a plan, asks for approval, and creates/updates/deletes resources to match the desired state.

```
Plan: 2 to create

+ Bucket (Cloudflare.R2.Bucket)
+ Worker (Cloudflare.Worker) (1 bindings)
  + Bucket

Proceed?
◉ Yes ○ No
✓ Bucket (Cloudflare.R2.Bucket) created
✓ Worker (Cloudflare.Worker) created
  • Uploading worker (14.20 KB) ...
  • Enabling workers.dev subdomain...
{
  url: "https://myapp-worker-dev-you-abc123.workers.dev",
}
```

On subsequent deploys, only changed resources are updated:

```
Plan: 1 to update

~ Worker (Cloudflare.Worker)

Proceed?
◉ Yes ○ No
✓ Worker (Cloudflare.Worker) updated
  • Uploading worker (15.10 KB) ...
{
  url: "https://myapp-worker-dev-you-abc123.workers.dev",
}
```

When the plan contains no changes, the approval prompt is skipped automatically.

## Flags

| Option | Description |
| --- | --- |
| `[file]` | Stack file to deploy (defaults to `alchemy.run.ts`) |
| `--stage <name>` | Stage to deploy to (defaults to `dev_$USER`) |
| `--yes` | Skip the approval prompt |
| `--dry-run` | Show the plan without applying (same as `alchemy plan`) |
| `--force` | Force updates for resources that would otherwise no-op |
| `--adopt` | Adopt pre-existing cloud resources that conflict with this stack instead of failing. Useful for re-importing into a fresh state store. |
| `--profile <name>` | Auth profile to use (defaults to `default` or `$ALCHEMY_PROFILE`) |
| `--env-file <path>` | Load environment variables from a file |

How `--adopt` decides what to take over is covered in [Adopting Resources](adopting-resources.md).

`--dry-run` runs the exact same code path as [`alchemy plan`](plan.md) — see that page for the plan output format.

## Examples

```sh
# Deploy to production, skip the prompt
alchemy deploy --stage prod --yes

# Deploy a PR preview environment
alchemy deploy --stage pr-42

# Deploy a different stack file
alchemy deploy stacks/github.ts

# Preview what would change
alchemy deploy --dry-run

# Re-import existing cloud resources into a fresh state store
alchemy deploy --adopt
```

## Where next

- [plan](plan.md) — preview changes without applying
- [destroy](destroy.md) — delete everything in a stack
- [Adopting Resources](adopting-resources.md) — take over pre-existing infrastructure
- [Resource Lifecycle](../infrastructure-as-code/resource-lifecycle.md) — what happens inside a deploy
