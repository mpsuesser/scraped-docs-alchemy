---
url: https://alchemy.run/e2b/snapshots
title: "Snapshots & forks"
description: "Start sandboxes from a warm base, snapshot a running sandbox, and fork one into copies."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **snapshot** saves a sandbox, memory and disk, as a template of its own. Sandboxes from it start where the original was: a repository cloned, its dependencies installed, a server listening. A **fork** snapshots a running sandbox and starts copies of it at once, while the original keeps running. Each copy changes on its own from then on.

## Copies start where the original was

<svg viewBox="0 0 656 320" style="max-width: 754px" role="img" aria-label="Warm is MyTemplate after npm ci. user-42 starts from Warm, and two forks copy user-42 as it is, edits included, to run test shards side by side."><g><text fill="currentColor" x="4" y="30">MyTemplate</text> <text fill="currentColor" x="4" y="44">template</text> <line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="30" x2="640" y2="30"></line></g><g><text fill="currentColor" x="4" y="94">Warm</text> <text fill="currentColor" x="4" y="108">snapshot</text> <path stroke="currentColor" fill="none" d="M 154 30 C 172 30, 162 94, 180 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="180" y1="94" x2="640" y2="94"></line></g><g><text fill="currentColor" x="4" y="158">user-42</text> <text fill="currentColor" x="4" y="172">sandbox</text> <path stroke="currentColor" fill="none" d="M 262 94 C 280 94, 270 158, 288 158"></path><line stroke-opacity="0.2" stroke="currentColor" x1="288" y1="158" x2="640" y2="158" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="222">fork 1</text> <text fill="currentColor" x="4" y="236">sandbox</text> <path stroke="currentColor" fill="none" d="M 424 158 C 442 158, 432 222, 450 222"></path><line stroke-opacity="0.2" stroke="currentColor" x1="450" y1="222" x2="586" y2="222"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="581" y1="217" x2="591" y2="227"></line><line stroke-opacity="0.2" stroke="currentColor" x1="581" y1="227" x2="591" y2="217"></line></g></g><g><text fill="currentColor" x="4" y="286">fork 2</text> <text fill="currentColor" x="4" y="300">sandbox</text> <path stroke="currentColor" fill="none" d="M 424 158 C 442 158, 432 286, 450 286"></path><line stroke-opacity="0.2" stroke="currentColor" x1="450" y1="286" x2="586" y2="286"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="581" y1="281" x2="591" y2="291"></line><line stroke-opacity="0.2" stroke="currentColor" x1="581" y1="291" x2="591" y2="281"></line></g></g><g><circle stroke="currentColor" fill="none" cx="208" cy="94" r="5.5"></circle><text fill="currentColor" x="208" y="83" text-anchor="middle">npm ci</text></g> <g><circle stroke="currentColor" fill="none" cx="343" cy="158" r="5.5"></circle><text fill="currentColor" x="343" y="147" text-anchor="middle">edit files</text></g> <g><circle stroke="currentColor" fill="none" cx="505" cy="222" r="5.5"></circle><text fill="currentColor" x="505" y="211" text-anchor="middle">test shard 1</text></g> <g><circle stroke="currentColor" fill="none" cx="505" cy="286" r="5.5"></circle><text fill="currentColor" x="505" y="275" text-anchor="middle">test shard 2</text></g></svg> <svg viewBox="0 0 328 392" style="max-width: 377px" role="img" aria-label="Warm is MyTemplate after npm ci. user-42 starts from Warm, and two forks copy user-42 as it is, edits included, to run test shards side by side."><g><text fill="currentColor" x="0" y="16">MyTemplate <tspan fill="currentColor" dx="8">template</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46"></line></g><g><text fill="currentColor" x="64.4" y="94">Warm <tspan fill="currentColor" dx="8">snapshot</tspan></text> <path stroke="currentColor" fill="none" d="M 38.400000000000006 46 C 56.400000000000006 46, 46.400000000000006 124, 64.4 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="64.4" y1="124" x2="312" y2="124"></line></g><g><text fill="currentColor" x="125.2" y="172">user-42 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <path stroke="currentColor" fill="none" d="M 99.2 124 C 117.2 124, 107.2 202, 125.2 202"></path><line stroke-opacity="0.2" stroke="currentColor" x1="125.2" y1="202" x2="312" y2="202" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="216.4" y="250">fork 1 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <path stroke="currentColor" fill="none" d="M 190.4 202 C 208.4 202, 198.4 280, 216.4 280"></path><line stroke-opacity="0.2" stroke="currentColor" x1="216.4" y1="280" x2="281.6" y2="280"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="275" x2="286.6" y2="285"></line><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="285" x2="286.6" y2="275"></line></g></g><g><text fill="currentColor" x="216.4" y="328">fork 2 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <path stroke="currentColor" fill="none" d="M 190.4 202 C 208.4 202, 198.4 358, 216.4 358"></path><line stroke-opacity="0.2" stroke="currentColor" x1="216.4" y1="358" x2="281.6" y2="358"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="353" x2="286.6" y2="363"></line><line stroke-opacity="0.2" stroke="currentColor" x1="276.6" y1="363" x2="286.6" y2="353"></line></g></g><g><circle stroke="currentColor" fill="none" cx="68.80000000000001" cy="124" r="5.5"></circle><text fill="currentColor" x="49.60000000000001" y="113" text-anchor="start">npm ci</text></g> <g><circle stroke="currentColor" fill="none" cx="144.8" cy="202" r="5.5"></circle><text fill="currentColor" x="112.80000000000001" y="191" text-anchor="start">edit files</text></g> <g><circle stroke="currentColor" fill="none" cx="236" cy="280" r="5.5"></circle><text fill="currentColor" x="197.6" y="269" text-anchor="start">test shard 1</text></g> <g><circle stroke="currentColor" fill="none" cx="236" cy="358" r="5.5"></circle><text fill="currentColor" x="197.6" y="347" text-anchor="start">test shard 2</text></g></svg>

Warm is MyTemplate after npm ci. user-42 starts from Warm, and two forks copy user-42 as it is, edits included, to run test shards side by side.

## Build a warm base at deploy

```typescript
const warm = yield* E2B.Snapshot("Warm", {
  template: MyTemplate,
  run: [
    "git clone https://github.com/my-org/app /home/user/app",
    "cd /home/user/app && npm ci",
  ],
});
```

Alchemy boots the template, runs the commands, snapshots the sandbox with its memory, and kills it. A command that exits non-zero fails the deploy with `SnapshotSetupFailed`. A deploy runs the setup again only when the template or the commands change.

## Start sandboxes from a snapshot

```typescript
const sandboxes = yield* E2B.CreateSandbox(Warm);
const sandbox = yield* sandboxes.create({ timeout: 600 });
yield* sandbox.exec("cd app && npm test");
```

## Snapshot a running sandbox

```typescript
const snapshot = yield* sandbox.snapshot({ name: \`session-${id}\` });
const restored = yield* sandboxes.create({ templateID: snapshot.snapshotID });
```

Pass `memory: false` to keep only the filesystem; sandboxes from it boot fresh.

## Fork a running sandbox

```typescript
const results = yield* sandbox.fork({ count: 3 });
for (const result of results) {
  if (result._tag === "Forked") yield* result.sandbox.exec("npm test -- --shard");
}
```

E2B checkpoints the sandbox (it keeps running) and starts copies of it. Each fork succeeds or fails on its own.

## Take snapshots while your app runs

```typescript
const snapshots = yield* E2B.CreateSnapshot();
const info = yield* snapshots.create(\`warm-${slug}\`, { template: "node-22", run: ["npm ci"] });
yield* snapshots.delete(\`warm-${slug}\`);
```

Runtime snapshots are not deleted when you destroy the Worker.
