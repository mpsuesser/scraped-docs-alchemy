---
url: https://alchemy.run/boat/sandboxes/snapshots
title: "Snapshots & forks"
description: "Save a Boat sandbox's disk as a named snapshot, start sandboxes from it, and fork a sandbox into copies."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **snapshot** is a sandbox’s disk saved under a name: installed packages, a cloned repository, a warm build cache. A sandbox started from it begins exactly there. A **fork** copies one sandbox into a new one from its last snapshot, which Boat takes now and then while it runs, and every time it stops.

## Every sandbox starts from the same disk

The setup runs once, at deploy. Every sandbox after it starts with the work already done, and changes only its own copy:

<svg viewBox="0 0 695.2 192" style="max-width: 799px" role="img" aria-label="Alchemy runs the snapshot's commands in a sandbox of its own, saves it as Golden and deletes it. Every task sandbox starts from Golden, with the setup already done, and changes only its own copy."><g><text fill="currentColor" x="4" y="30">setup sandbox</text> <text fill="currentColor" x="4" y="44">deleted once saved</text> <line stroke-opacity="0.2" stroke="currentColor" x1="139.2" y1="30" x2="355.19999999999993" y2="30"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="350.19999999999993" y1="25" x2="360.19999999999993" y2="35"></line><line stroke-opacity="0.2" stroke="currentColor" x1="350.19999999999993" y1="35" x2="360.19999999999993" y2="25"></line></g></g><g><text fill="currentColor" x="4" y="94">task-1</text> <text fill="currentColor" x="4" y="108">from Golden</text> <path stroke="currentColor" fill="none" d="M 319.2 30 C 337.2 30, 327.2 94, 345.2 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="345.2" y1="94" x2="679.2" y2="94" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="158">task-2</text> <text fill="currentColor" x="4" y="172">from Golden</text> <path stroke="currentColor" fill="none" d="M 319.2 30 C 337.2 30, 327.2 158, 345.2 158"></path><line stroke-opacity="0.2" stroke="currentColor" x1="345.2" y1="158" x2="679.2" y2="158" style="stroke-width: 3"></line></g><g><circle stroke="currentColor" fill="none" cx="229.2" cy="30" r="5.5"></circle><text fill="currentColor" x="229.2" y="19" text-anchor="middle">run</text></g> <g><circle stroke="currentColor" fill="none" cx="319.2" cy="30" r="5.5"></circle><text fill="currentColor" x="319.2" y="19" text-anchor="middle">Golden</text></g> <g><circle stroke="currentColor" fill="none" cx="499.2" cy="94" r="5.5"></circle><text fill="currentColor" x="499.2" y="83" text-anchor="middle">agent edits</text></g></svg> <svg viewBox="0 0 328 236" style="max-width: 377px" role="img" aria-label="Alchemy runs the snapshot's commands in a sandbox of its own, saves it as Golden and deletes it. Every task sandbox starts from Golden, with the setup already done, and changes only its own copy."><g><text fill="currentColor" x="0" y="16">setup sandbox <tspan fill="currentColor" dx="8">deleted once saved</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="129.6" y2="46"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="124.6" y1="41" x2="134.6" y2="51"></line><line stroke-opacity="0.2" stroke="currentColor" x1="124.6" y1="51" x2="134.6" y2="41"></line></g></g><g><text fill="currentColor" x="135.33333333333331" y="94">task-1 <tspan fill="currentColor" dx="8">from Golden</tspan></text> <path stroke="currentColor" fill="none" d="M 109.33333333333333 46 C 127.33333333333333 46, 117.33333333333333 124, 135.33333333333331 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="135.33333333333331" y1="124" x2="312" y2="124" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="135.33333333333331" y="172">task-2 <tspan fill="currentColor" dx="8">from Golden</tspan></text> <path stroke="currentColor" fill="none" d="M 109.33333333333333 46 C 127.33333333333333 46, 117.33333333333333 202, 135.33333333333331 202"></path><line stroke-opacity="0.2" stroke="currentColor" x1="135.33333333333331" y1="202" x2="312" y2="202" style="stroke-width: 3"></line></g><g><circle stroke="currentColor" fill="none" cx="58.666666666666664" cy="46" r="5.5"></circle><text fill="currentColor" x="49.06666666666666" y="35" text-anchor="start">run</text></g> <g><circle stroke="currentColor" fill="none" cx="109.33333333333333" cy="46" r="5.5"></circle><text fill="currentColor" x="90.13333333333333" y="35" text-anchor="start">Golden</text></g> <g><circle stroke="currentColor" fill="none" cx="210.66666666666666" cy="124" r="5.5"></circle><text fill="currentColor" x="175.46666666666664" y="113" text-anchor="start">agent edits</text></g></svg>

Alchemy runs the snapshot's commands in a sandbox of its own, saves it as Golden and deletes it. Every task sandbox starts from Golden, with the setup already done, and changes only its own copy.

## Build a snapshot from commands

List the commands that set it up:

```typescript
import * as Boat from "@alchemy.run/boat";

export const Golden = Boat.Snapshot("Golden", {
  run: [
    "git clone https://github.com/alchemy-run/alchemy ~/alchemy",
    "cd ~/alchemy && pnpm install",
  ],
});
```

Alchemy starts a sandbox with none of your account’s secrets, runs the commands in order, saves its disk and deletes the sandbox. A command that exits non-zero fails the deploy with `SnapshotSetupFailed`. A deploy runs them again only when the commands, `env`, `cwd`, `type` or `from` change, and saves the new disk under the same name. Destroying the resource deletes it.

Build one snapshot on another with `from`:

```typescript
export const Tools = Boat.Snapshot("Tools", { run: ["npm i -g pnpm"] });
export const Golden = Boat.Snapshot("Golden", { from: Tools, run: ["pnpm dlx playwright install"] });
```

## Start sandboxes from a snapshot

A declared sandbox starts from a snapshot with `snapshot`:

```typescript
export const Box = Boat.Sandbox("Box", { snapshot: Golden });
```

When the snapshot is rebuilt, the sandbox is replaced, so it starts from the new disk.

So do the sandboxes a binding creates, when you bind the snapshot:

```typescript
const sandboxes = yield* Boat.CreateSandbox(Golden);
const sandbox = yield* sandboxes.create({ autoStop: "10 minutes" });
```

A sandbox from a snapshot takes its machine size unless you pass `type`.

## Fork a sandbox into a copy

`fork` starts a copy of a sandbox your code created, from its last snapshot:

```typescript
yield* sandbox.stop(); // stopping takes a snapshot
const copy = yield* sandbox.fork({ autoStop: "10 minutes" });
yield* sandbox.resume();
```

Before a sandbox has a snapshot, `fork` fails with `ForkNeedsSnapshot`. Boat never copies labels onto a fork; Alchemy adds the host’s `alchemy.*` labels again.

## Save a running sandbox under a name

A handle saves its sandbox under a name of your own, and waits until it’s ready:

```typescript
const saved = yield* sandbox.snapshot("after-install");
```

Snapshots your code saves aren’t deleted with the Stack.

To give sandboxes variables, secret files and credentials, see [Environments & secrets](../security/environments.md).
