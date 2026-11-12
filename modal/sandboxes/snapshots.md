---
url: https://alchemy.run/modal/sandboxes/snapshots
title: "Snapshots"
description: "Save a Modal sandbox's files, its memory, or one directory, and start new sandboxes from it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **snapshot** saves a running sandbox so another can start where it was. Modal saves three things: the whole filesystem as an image, the memory with its running processes, or one directory to mount elsewhere.

## A copy starts where the original was

The original keeps running after a snapshot. Every copy starts from the saved state, and changes only its own files:

<svg viewBox="0 0 688.8 128" style="max-width: 792px" role="img" aria-label="session-sam installs dependencies and takes a snapshot. The copy starts with them installed, and the agent's later edits stay in session-sam."><g><text fill="currentColor" x="4" y="30">session-sam</text> <text fill="currentColor" x="4" y="44">sandbox</text> <line stroke-opacity="0.2" stroke="currentColor" x1="132.8" y1="30" x2="672.8" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">copy</text> <text fill="currentColor" x="4" y="108">from the snapshot</text> <path stroke="currentColor" fill="none" d="M 312.8 30 C 330.8 30, 320.8 94, 338.8 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="338.8" y1="94" x2="672.8" y2="94"></line></g><g><circle stroke="currentColor" fill="none" cx="222.8" cy="30" r="5.5"></circle><text fill="currentColor" x="222.8" y="19" text-anchor="middle">pnpm install</text></g> <g><circle stroke="currentColor" fill="none" cx="312.8" cy="30" r="5.5"></circle><text fill="currentColor" x="312.8" y="19" text-anchor="middle">snapshot</text></g> <g><circle stroke="currentColor" fill="none" cx="492.8" cy="30" r="5.5"></circle><text fill="currentColor" x="492.8" y="19" text-anchor="middle">agent edits</text></g> <g><circle stroke="currentColor" fill="none" cx="537.8" cy="94" r="5.5"></circle><text fill="currentColor" x="537.8" y="83" text-anchor="middle">pnpm test</text></g></svg> <svg viewBox="0 0 328 158" style="max-width: 377px" role="img" aria-label="session-sam installs dependencies and takes a snapshot. The copy starts with them installed, and the agent's later edits stay in session-sam."><g><text fill="currentColor" x="0" y="16">session-sam <tspan fill="currentColor" dx="8">sandbox</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="135.33333333333331" y="94">copy <tspan fill="currentColor" dx="8">from the snapshot</tspan></text> <path stroke="currentColor" fill="none" d="M 109.33333333333333 46 C 127.33333333333333 46, 117.33333333333333 124, 135.33333333333331 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="135.33333333333331" y1="124" x2="312" y2="124"></line></g><g><circle stroke="currentColor" fill="none" cx="58.666666666666664" cy="46" r="5.5"></circle><text fill="currentColor" x="20.26666666666666" y="35" text-anchor="start">pnpm install</text></g> <g><circle stroke="currentColor" fill="none" cx="109.33333333333333" cy="46" r="5.5"></circle><text fill="currentColor" x="83.73333333333332" y="67" text-anchor="start">snapshot</text></g> <g><circle stroke="currentColor" fill="none" cx="210.66666666666666" cy="46" r="5.5"></circle><text fill="currentColor" x="175.46666666666664" y="35" text-anchor="start">agent edits</text></g> <g><circle stroke="currentColor" fill="none" cx="236" cy="124" r="5.5"></circle><text fill="currentColor" x="207.2" y="113" text-anchor="start">pnpm test</text></g></svg>

session-sam installs dependencies and takes a snapshot. The copy starts with them installed, and the agent's later edits stay in session-sam.

## Save the filesystem as an image

```typescript
yield* sandbox.exec(["pnpm", "install"], { cwd: "/work" });
const image = yield* sandbox.snapshotFilesystem();
```

The result is an image. Pass it to `create`:

```typescript
const next = yield* sandboxes.create({ image });
```

## Save memory and running processes

A sandbox created with `enableSnapshot` can save its memory too. The copy resumes with its processes running:

```typescript
const sandbox = yield* sandboxes.create({ enableSnapshot: true });
const snapshot = yield* sandbox.snapshotMemory();
const restored = yield* sandboxes.restore(snapshot, { name: \`session-${user}\` });
```

## Move one directory to another sandbox

```typescript
const workspace = yield* sandbox.snapshotDirectory("/work");
yield* other.mountDirectory("/work", workspace);
```

`other` gets a copy of `/work` and nothing else from the original.
