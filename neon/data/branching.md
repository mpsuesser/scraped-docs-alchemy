---
url: https://alchemy.run/neon/data/branching
title: "Branching"
description: "Neon branches are copy-on-write forks of a parent branch — create them per stage or preview, pin them to a point in time, copy schema only, and let them expire on their own."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A `Neon.Branch` is a copy-on-write fork of a parent branch. It shares storage with the parent until it starts diverging, so creating one takes seconds regardless of database size — cheap enough to fork a branch per stage, per pull request, or per experiment.

<svg viewBox="0 0 644 192" style="max-width: 741px" role="img" aria-label="A branch starts as a copy-on-write fork, dashed while it still shares its parent's storage and solid once it writes its own. A preview branch goes away with its stage."><g><text fill="currentColor" x="4" y="30">main</text> <text fill="currentColor" x="4" y="44">production</text> <line stroke-opacity="0.2" stroke="currentColor" x1="88" y1="30" x2="628" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">pr-42</text> <text fill="currentColor" x="4" y="108">preview</text> <path stroke="currentColor" fill="none" d="M 223 30 C 241 30, 231 94, 249 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="249" y1="94" x2="358" y2="94"></line><line stroke-opacity="0.2" stroke="currentColor" x1="358" y1="94" x2="520" y2="94"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="515" y1="89" x2="525" y2="99"></line><line stroke-opacity="0.2" stroke="currentColor" x1="515" y1="99" x2="525" y2="89"></line></g></g><g><text fill="currentColor" x="4" y="158">pr-43</text> <text fill="currentColor" x="4" y="172">preview</text> <path stroke="currentColor" fill="none" d="M 412 30 C 430 30, 420 158, 438 158"></path><line stroke-opacity="0.2" stroke="currentColor" x1="438" y1="158" x2="574" y2="158"></line><line stroke-opacity="0.2" stroke="currentColor" x1="574" y1="158" x2="628" y2="158"></line></g><g><circle stroke="currentColor" fill="none" cx="142" cy="30" r="5.5"></circle><text fill="currentColor" x="142" y="19" text-anchor="middle">orders table</text></g> <g><circle stroke="currentColor" fill="none" cx="358" cy="94" r="5.5"></circle><text fill="currentColor" x="358" y="83" text-anchor="middle">first write</text></g> <g><circle stroke="currentColor" fill="none" cx="466" cy="30" r="5.5"></circle><text fill="currentColor" x="466" y="19" text-anchor="middle">new rows</text></g> <g><circle stroke="currentColor" fill="none" cx="520" cy="94" r="5.5"></circle><text fill="currentColor" x="520" y="115" text-anchor="middle">stage destroyed</text></g></svg> <svg viewBox="0 0 328 236" style="max-width: 377px" role="img" aria-label="A branch starts as a copy-on-write fork, dashed while it still shares its parent's storage and solid once it writes its own. A preview branch goes away with its stage."><g><text fill="currentColor" x="0" y="16">main <tspan fill="currentColor" dx="8">production</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="110" y="94">pr-42 <tspan fill="currentColor" dx="8">preview</tspan></text> <path stroke="currentColor" fill="none" d="M 84 46 C 102 46, 92 124, 110 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="110" y1="124" x2="160" y2="124"></line><line stroke-opacity="0.2" stroke="currentColor" x1="160" y1="124" x2="251.20000000000002" y2="124"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="246.20000000000002" y1="119" x2="256.20000000000005" y2="129"></line><line stroke-opacity="0.2" stroke="currentColor" x1="246.20000000000002" y1="129" x2="256.20000000000005" y2="119"></line></g></g><g><text fill="currentColor" x="216.4" y="172">pr-43 <tspan fill="currentColor" dx="8">preview</tspan></text> <path stroke="currentColor" fill="none" d="M 190.4 46 C 208.4 46, 198.4 202, 216.4 202"></path><line stroke-opacity="0.2" stroke="currentColor" x1="216.4" y1="202" x2="281.6" y2="202"></line><line stroke-opacity="0.2" stroke="currentColor" x1="281.6" y1="202" x2="312" y2="202"></line></g><g><circle stroke="currentColor" fill="none" cx="38.400000000000006" cy="46" r="5.5"></circle><text fill="currentColor" x="2" y="35" text-anchor="start">orders table</text></g> <g><circle stroke="currentColor" fill="none" cx="160" cy="124" r="5.5"></circle><text fill="currentColor" x="124.8" y="113" text-anchor="start">first write</text></g> <g><circle stroke="currentColor" fill="none" cx="220.79999999999998" cy="46" r="5.5"></circle><text fill="currentColor" x="195.2" y="35" text-anchor="start">new rows</text></g> <g><circle stroke="currentColor" fill="none" cx="251.20000000000002" cy="124" r="5.5"></circle><text fill="currentColor" x="203.20000000000002" y="145" text-anchor="start">stage destroyed</text></g></svg>

A branch starts as a copy-on-write fork, dashed while it still shares its parent's storage and solid once it writes its own. A preview branch goes away with its stage.

## Create a branch

```typescript
const project = yield* Neon.Project("my-project");
const dev = yield* Neon.Branch("dev-branch", { project });
```

With no other props, the branch forks from the project’s default branch and gets a unique name generated from the app, stage, and logical ID. Pass `name` to control it. The outputs include `connectionUri`, `pooledConnectionUri`, and parsed `origin` / `pooledOrigin` components — see [Connections](connections.md) for how to use them.

## Choose the parent

`parentBranch` selects which branch to fork from:

```typescript
const dev = yield* Neon.Branch("dev", { project });
const featureBranch = yield* Neon.Branch("feature", {
  project,
  parentBranch: dev,
});
```

It accepts another `Branch` resource, a `{ branchId: "br-..." }`, or a `{ name: "dev" }` to look up by name — the deploy fails if the named branch is missing or the name is ambiguous. When omitted, the project’s default branch is the parent.

## Branch from a point in time

`parentLsn` forks the parent’s data as of a specific Log Sequence Number:

```typescript
const branch = yield* Neon.Branch("at-lsn", {
  project,
  parentLsn: "0/3FA01B0",
});
```

`parentTimestamp` does the same with an ISO-8601 timestamp:

```typescript
const branch = yield* Neon.Branch("before-incident", {
  project,
  parentTimestamp: "2026-06-30T12:00:00Z",
});
```

The fork point is fixed at creation, so changing `parentLsn` or `parentTimestamp` on a subsequent deploy replaces the branch with a fresh fork.

## Schema-only branches

`initSource: "schema-only"` copies the parent’s schema without its data — useful when the parent holds data that shouldn’t leak into development branches:

```typescript
const schemaBranch = yield* Neon.Branch("schema-only", {
  project,
  initSource: "schema-only",
});
```

The default is `"parent-data"` (schema and data). Like the fork point, `initSource` is decided at creation — changing it replaces the branch.

## Expiring preview branches

`expiresAt` tells Neon to delete the branch itself at an RFC-3339 timestamp — a safety net for ephemeral preview branches that might outlive their deploy:

```typescript
const preview = yield* Neon.Branch("preview", {
  project,
  expiresAt: "2026-07-08T00:00:00Z",
});
```

Unlike the fork point, `expiresAt` is mutable: changing or removing it updates the branch in place. The inverse guard is `protected: true`, which marks the branch protected from deletion and mutation.

## Compute endpoints

By default each branch gets one `read_write` compute endpoint — required to connect at all. `endpoints` overrides the default with explicit compute settings:

```typescript
const branch = yield* Neon.Branch("analytics", {
  project,
  endpoints: [
    {
      type: "read_write",
      autoscalingLimitMinCu: 0.25,
      autoscalingLimitMaxCu: 2,
      suspendTimeoutSeconds: 300,
    },
    { type: "read_only" },
  ],
});
```

`autoscalingLimitMinCu` / `autoscalingLimitMaxCu` bound the compute size the endpoint scales between, and `suspendTimeoutSeconds` controls how long the endpoint idles before suspending.

## Where next

Guides:

- [Branch from a shared database](../../cloudflare/data/branch-from-shared-database.md) — fork a preview branch per stage off a long-lived staging project.

Related:

- [Connections](connections.md) — direct vs pooled URIs and feeding `origin` into Hyperdrive.
- [Migrations](migrations.md) — apply SQL migrations on the branch as part of the deploy.

Reference:

- [Branch API reference](https://alchemy.run/providers/neon/reference/branch#branch)
