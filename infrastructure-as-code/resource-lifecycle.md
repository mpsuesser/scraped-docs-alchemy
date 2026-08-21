---
url: https://alchemy.run/infrastructure-as-code/resource-lifecycle
title: "Resource lifecycle"
description: "How alchemy plans, applies, replaces, and destroys resources — and how to think about idempotency and recovery."
access_date: 2026-08-21T19:05:43.655Z
current_date: 2026-08-21T19:05:43.655Z
---

Every [Resource](resource.md) goes through the same lifecycle: **plan → reconcile → (replace) → delete**. The plan classifies each resource as create, update, replace, delete, or no-op, but the provider implements a single `reconcile` function that converges the cloud’s actual state to what’s declared — whether that’s the first provisioning, a routine update, or an adoption takeover. For the CLI flags that drive these operations, see the [CLI reference](../cli.md).

## The lifecycle, end-to-end

<svg viewBox="0 0 628 356" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 178 C 201 178, 201 52, 238 52" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="201" y="109" text-anchor="middle">new</text></g> <g><path d="M 164 178 C 201 178, 201 136, 238 136" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="201" y="151" text-anchor="middle">changed</text></g> <g><path d="M 164 178 C 201 178, 201 220, 238 220" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="201" y="193" text-anchor="middle">breaking</text></g> <g><path d="M 164 178 C 311 178, 311 178, 458 178" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="311" y="172" text-anchor="middle">removed</text></g> <g><path d="M 164 178 C 201 178, 201 304, 238 304" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="201" y="235" text-anchor="middle">unchanged</text></g> <g><path d="M 384 220 C 421 220, 421 178, 458 178" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 150)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Plan</text></g> <g transform="translate(244, 24)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Reconcile (create)</text></g> <g transform="translate(244, 108)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Reconcile (update)</text></g> <g transform="translate(244, 192)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Replace</text></g> <g transform="translate(464, 150)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Delete</text></g> <g transform="translate(244, 276)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Noop</text></g></svg>

Both create and update intents resolve to the provider’s single `reconcile` function. Replace runs `reconcile` against a fresh instance id and then deletes the old generation. The same engine drives `alchemy deploy`, `alchemy destroy`, and `alchemy dev`.

## Plan

When you run `alchemy deploy`, alchemy first **plans** the change. It compares the desired state (what your code declares) against the last persisted state and classifies each resource:

- **Create** (`+`) — declared in code, not in state
- **Update** (`~`) — declared and persisted, but properties differ
- **Replace** (`±`) — change requires destroy-and-recreate
- **Delete** (`-`) — persisted but no longer declared
- **No-op** (`•`) — unchanged

```
Plan: 1 to create, 1 to update

+ Queue (AWS.SQS.Queue)
~ Worker (Cloudflare.Worker)
• Bucket (Cloudflare.R2.Bucket)
```

The classification comes from each provider’s `diff` function. See [Provider › diff](provider.md#diff) for how providers decide between in-place updates and replacements.

Use `alchemy plan` (or `alchemy deploy --dry-run`) to see the plan without applying it.

## Reconcile

Whether a resource is being created for the first time, updated in place, or adopted from existing infrastructure, alchemy calls a single function: `provider.reconcile`.

Reconcile must be **convergent**: given the desired state in `news`, it brings the cloud to that state regardless of starting point. It receives:

- `news` — desired props
- `output` — current attributes (`undefined` on greenfield, defined after a prior reconcile or after adoption)
- `olds` — previous props (`undefined` on greenfield AND on adoption; defined only on routine updates)
- `bindings` — resolved binding payload from upstream policies

A reconciler is shaped like **observe → ensure → sync → return**: read live cloud state via `getX` / `describeX`, create the resource if missing (catching `AlreadyExists` -style errors as races), then for each mutable aspect diff observed cloud state against desired and apply only the delta.

Because each step is independently idempotent, a partial reconcile that crashed midway resumes correctly on the next run. Physical names are deterministic from `stack/stage/logical-id`, so the “observe” step finds the previous reconcile’s output even if state persistence failed.

A second pass — **convergence** — re-runs `reconcile` for any resource whose inputs changed because an upstream output changed mid-deploy.

## Replace

Some property changes can’t be applied in place — for example, changing a DynamoDB table’s partition key. The provider’s `diff` returns `{ action: "replace" }`, and alchemy:

1. Creates a new resource with a new instance ID
2. Updates downstream resources to reference the new resource
3. Deletes the old resource

Because new and old coexist briefly, dependents get a clean cutover without downtime.

## Delete

`provider.delete` is called when a resource disappears from your code, when a replacement supersedes it, or when you run `alchemy destroy`. Like create, delete must be **idempotent**: deleting an already-gone resource is a success, not an error.

`alchemy destroy` is just a plan where every persisted resource is marked for deletion. Resources are removed in **reverse dependency order** — dependents go first.

```
Plan: 2 to delete

- Worker (Cloudflare.Worker)
- Bucket (Cloudflare.R2.Bucket)

Proceed?
◉ Yes ○ No
✗ Worker (Cloudflare.Worker) deleted
✗ Bucket (Cloudflare.R2.Bucket) deleted
```

## Removal policy

Each resource carries a **removal policy** that decides what happens to the physical cloud object when the resource is deleted — orphaned, destroyed, or superseded as a replacement’s old generation:

| Policy | What the engine does |
| --- | --- |
| `destroy` | Calls `provider.delete`, then drops the state row |
| `retain` | Skips `provider.delete`, then drops the state row |

Under `retain`, alchemy forgets the resource either way — only the cloud object survives. Most resource types default to `destroy`; a few whose contents are irreplaceable default to `retain` (e.g. [`GitHub.Repository`](../github/repository.md), `Cloudflare.Zone`).

Set the policy by piping a declaration through `retain()` or `destroy()`. It applies to every resource declared inside the piped effect, so it can decorate one resource or a whole scope:

```typescript
import * as RemovalPolicy from "alchemy/RemovalPolicy";

Effect.gen(function* () {
  const stack = yield* Stack;

  // never deleted by alchemy
  const uploads = yield* R2.Bucket("Uploads").pipe(RemovalPolicy.retain());

  // retained in prod, torn down in every other stage
  const cache = yield* R2.Bucket("Cache").pipe(
    RemovalPolicy.retain(stack.stage === "prod"),
  );
});
```

The policy is a decoration, not a prop, so changing it produces no diff — the resource plans as a **no-op** and the plan reports no changes. The deploy still persists the new policy onto the state row (the orphan delete reads it from there, long after the declaration is gone), so a policy change takes effect from the very next deploy.

## Idempotency and recovery

State persistence can fail after the cloud operation succeeds — the network drops between “bucket created” and “state saved”. Alchemy handles this by requiring `reconcile` and `delete` to be safe to retry:

- **Reconcile**: deterministic physical names plus the observe-step mean a retry finds the existing resource instead of creating a duplicate, and re-syncs any aspect that drifted.
- **Delete**: a missing resource is treated as already deleted.
- **Read**: providers can implement `read` so alchemy can recover state from the live cloud when persistence fails partway, and to detect adoptable resources on a fresh state store.

## Adoption

When planning a resource that has no prior state, the engine calls `provider.read` (if the provider implements it). This serves two overlapping purposes:

- **State recovery** — the resource was created on a previous deploy, but state was lost between the cloud op succeeding and the store persisting. `read` finds the live resource and the engine rebuilds `created` state from its attributes.
- **Adoption** — you’re deploying against existing infrastructure you didn’t manage with Alchemy yet (or you wiped state intentionally). `read` recognizes the resource and the engine imports it into the new state.

Providers signal “this is mine” vs. “this exists but isn’t mine” via the `Unowned(attrs)` brand. The engine routes:

| `read` returns | `--adopt` off | `--adopt` on |
| --- | --- | --- |
| `undefined` | create | create |
| owned (plain attrs) | silent adopt | silent adopt |
| `Unowned(attrs)` | fail `OwnedBySomeoneElse` | take over (silently) |

See [Provider › read](provider.md#read) for the implementation contract and [Adopting Resources](../cli/adopting-resources.md) for the CLI flag.

## Errors

- **Retryable errors** (eventual consistency, dependency races) are retried automatically with backoff.
- **Non-retryable errors** (validation, authorization) fail immediately and surface in the plan output.
- **Partial failures** are safe to re-run thanks to idempotency.

## Driving the lifecycle from the CLI

The same engine powers all of these commands:

| Command | What it does |
| --- | --- |
| `alchemy plan` | Run plan, print diff, exit |
| `alchemy deploy` | Plan, prompt for approval, apply |
| `alchemy destroy` | Plan with everything marked deleted, apply |
| `alchemy dev` | Plan + apply continuously on file changes |

See the [CLI reference](../cli.md) for the full set of flags (`--yes`, `--force`, `--dry-run`, `--stage`, `--profile`, …).

Every lifecycle operation on this page is implemented per resource type by a [Provider](provider.md).

## Where next

- [Providers](provider.md) — the object that implements `reconcile`, `delete`, `diff`, and `read` for a resource type.
- [CLI](../cli.md) — the commands and flags that drive the lifecycle.
- [State Store](../state-store.md) — where the persisted state behind the plan lives.
