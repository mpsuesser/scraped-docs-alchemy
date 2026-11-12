---
url: https://alchemy.run/archil/storage/branches
title: "Branches and checkpoints"
description: "Seed a disk once, checkpoint it at deploy time, and copy a writable branch per session in milliseconds."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **checkpoint** is a named snapshot of a disk. It takes about 15 ms whatever the disk’s size, and never changes once taken. A **branch** is a writable copy of a disk started from a checkpoint, in about 35 ms. Writes on a branch never reach the disk or its other branches, so each agent, session or CI job can get a copy of its own.

## Each branch shares the seed until it writes

<svg viewBox="0 0 656 192" style="max-width: 754px" role="img" aria-label="Both sessions start from the checkpoint base. Each branch shares the seed's data until it writes, and its writes stay on that branch."><g><text fill="currentColor" x="4" y="30">Seed</text> <text fill="currentColor" x="4" y="44">disk</text> <line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="30" x2="640" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">session-42</text> <text fill="currentColor" x="4" y="108">branch</text> <path stroke="currentColor" fill="none" d="M 262 30 C 280 30, 270 94, 288 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="288" y1="94" x2="370" y2="94"></line><line stroke-opacity="0.2" stroke="currentColor" x1="370" y1="94" x2="640" y2="94"></line></g><g><text fill="currentColor" x="4" y="158">session-43</text> <text fill="currentColor" x="4" y="172">branch</text> <path stroke="currentColor" fill="none" d="M 262 30 C 280 30, 270 158, 288 158"></path><line stroke-opacity="0.2" stroke="currentColor" x1="288" y1="158" x2="478" y2="158"></line><line stroke-opacity="0.2" stroke="currentColor" x1="478" y1="158" x2="640" y2="158"></line></g><g><circle stroke="currentColor" fill="none" cx="154" cy="30" r="5.5"></circle><text fill="currentColor" x="154" y="19" text-anchor="middle">git clone</text></g> <g><circle stroke="currentColor" fill="none" cx="262" cy="30" r="5.5"></circle><text fill="currentColor" x="262" y="19" text-anchor="middle">checkpoint base</text></g> <g><circle stroke="currentColor" fill="none" cx="370" cy="94" r="5.5"></circle><text fill="currentColor" x="370" y="83" text-anchor="middle">edit src/</text></g> <g><circle stroke="currentColor" fill="none" cx="478" cy="158" r="5.5"></circle><text fill="currentColor" x="478" y="147" text-anchor="middle">npm test</text></g></svg> <svg viewBox="0 0 328 236" style="max-width: 377px" role="img" aria-label="Both sessions start from the checkpoint base. Each branch shares the seed's data until it writes, and its writes stay on that branch."><g><text fill="currentColor" x="0" y="16">Seed <tspan fill="currentColor" dx="8">disk</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="125.2" y="94">session-42 <tspan fill="currentColor" dx="8">branch</tspan></text> <path stroke="currentColor" fill="none" d="M 99.2 46 C 117.2 46, 107.2 124, 125.2 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="125.2" y1="124" x2="160" y2="124"></line><line stroke-opacity="0.2" stroke="currentColor" x1="160" y1="124" x2="312" y2="124"></line></g><g><text fill="currentColor" x="125.2" y="172">session-43 <tspan fill="currentColor" dx="8">branch</tspan></text> <path stroke="currentColor" fill="none" d="M 99.2 46 C 117.2 46, 107.2 202, 125.2 202"></path><line stroke-opacity="0.2" stroke="currentColor" x1="125.2" y1="202" x2="220.79999999999998" y2="202"></line><line stroke-opacity="0.2" stroke="currentColor" x1="220.79999999999998" y1="202" x2="312" y2="202"></line></g><g><circle stroke="currentColor" fill="none" cx="38.400000000000006" cy="46" r="5.5"></circle><text fill="currentColor" x="9.600000000000005" y="35" text-anchor="start">git clone</text></g> <g><circle stroke="currentColor" fill="none" cx="99.2" cy="46" r="5.5"></circle><text fill="currentColor" x="51.2" y="67" text-anchor="start">checkpoint base</text></g> <g><circle stroke="currentColor" fill="none" cx="160" cy="124" r="5.5"></circle><text fill="currentColor" x="131.2" y="113" text-anchor="start">edit src/</text></g> <g><circle stroke="currentColor" fill="none" cx="220.79999999999998" cy="202" r="5.5"></circle><text fill="currentColor" x="195.2" y="191" text-anchor="start">npm test</text></g></svg>

Both sessions start from the checkpoint base. Each branch shares the seed's data until it writes, and its writes stay on that branch.

## Seed a disk and checkpoint it

`Archil.Checkpoint` runs commands against the disk at deploy time, then takes the checkpoint in the same microVM, so it holds exactly what they produced:

```typescript
import * as Archil from "@alchemy.run/archil";
import * as Effect from "effect/Effect";

export const Seed = Archil.Disk("Seed", {});

export const Base = Effect.gen(function* () {
  const seed = yield* Seed;
  return yield* Archil.Checkpoint("Base", {
    disk: seed.diskId,
    name: "base",
    run: ["git clone --depth 1 https://github.com/acme/app repo"],
  });
});
```

## Branch a disk per session

`Archil.BranchDisk` gives a Worker or Lambda a client that creates branches while the app runs:

```typescript
const seeds = yield* Archil.BranchDisk(Seed);
const branch = yield* seeds.branch({ name: "session-42", fromCheckpoint: "base" });
```

Mount it on your own host with `MountDisk` and `branch: "session-42"`. Branches share the disk’s credentials, so a disk user that mounts the disk can mount any of its branches. A sandbox can’t mount a branch yet.

## Declare a long-lived branch

A branch that lives as long as the Stack, like a staging copy, is a resource:

```typescript
export const Staging = Effect.gen(function* () {
  const seed = yield* Seed;
  const base = yield* Base;
  return yield* Archil.Branch("Staging", { disk: seed.diskId, fromCheckpoint: base.name });
});
```

## Fork a sandbox in milliseconds

A sandbox’s root filesystem is a disk too, so `fork` uses the same checkpoint and branch to copy a sandbox in milliseconds. A sandbox with mounted disks has to be stopped first.

```typescript
const copy = yield* sandbox.fork({ name: "task-1-retry" });
```

## Branches go when the disk does

Archil can’t delete checkpoints or branches yet. Destroying an `Archil.Checkpoint` or `Archil.Branch` removes it from Alchemy’s state only, and deleting the disk deletes them with it.
