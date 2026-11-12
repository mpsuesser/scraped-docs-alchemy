---
url: https://alchemy.run/modal/sandboxes/persistent
title: "Persistent sandboxes"
description: "One Modal sandbox whose filesystem outlives its sessions, resumed from a Worker."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **persistent sandbox** is one sandbox whose files outlive the container that runs it. It runs in **sessions**: each one up to an hour, paused after five idle minutes, and resumed with the files the last one left.

## Sessions come and go, the files stay

<svg viewBox="0 0 711.2 80" style="max-width: 818px" role="img" aria-label="Session 1 pauses after five idle minutes. The next call to session() starts session 2 from the files session 1 left."><line stroke-opacity="0.2" stroke="currentColor" x1="307.2" y1="24" x2="307.2" y2="74"></line><line stroke-opacity="0.2" stroke="currentColor" x1="475.2" y1="24" x2="475.2" y2="74"></line><g><text fill="currentColor" x="4" y="50">DevBox</text> <text fill="currentColor" x="4" y="64">persistent sandbox</text> <g><rect stroke="currentColor" fill="none" x="139.2" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="223.2" y="56" text-anchor="middle">session 1</text></g> <g><rect stroke="currentColor" fill="none" x="307.2" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="391.2" y="56" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="475.2" y="40" width="224.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="587.2" y="56" text-anchor="middle">session 2</text></g></g> <text fill="currentColor" x="307.2" y="18" text-anchor="middle">5 idle minutes</text> <text fill="currentColor" x="475.2" y="18" text-anchor="middle">session()</text></svg> <svg viewBox="0 0 330 90" style="max-width: 379px" role="img" aria-label="Session 1 pauses after five idle minutes. The next call to session() starts session 2 from the files session 1 left."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="84"></line><line stroke-opacity="0.2" stroke="currentColor" x1="190.79999999999998" y1="24" x2="190.79999999999998" y2="84"></line><g><text fill="currentColor" x="0" y="44">DevBox <tspan fill="currentColor" dx="8">persistent sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="70" text-anchor="middle">session 1</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="70" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="54" width="127.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="70" text-anchor="middle">session 2</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">5 idle minutes</text> <text fill="currentColor" x="190.79999999999998" y="18" text-anchor="middle">session()</text></svg>

Session 1 pauses after five idle minutes. The next call to session() starts session 2 from the files session 1 left.

## Declare a box

```typescript
import * as Modal from "@alchemy.run/modal";
import { Coder, Warm } from "./Coder.ts";

export const DevBox = Modal.PersistentSandbox("DevBox", { app: Coder, image: Warm, workdir: "/work" });
```

## Resume it from a Worker

```typescript
// construction phase
const box = yield* Modal.ConnectSandbox(DevBox);
// in fetch:
const session = yield* box.session();
const status = yield* session.exec("git status");
```

`session()` returns the running session, or resumes the box from its last session’s files.

## Create one per user while your app runs

```typescript
const box = yield* sandboxes.createPersistent({ image: "node:22-bookworm" });
// store box.persistentSandboxId with the user
```

Persistent sandboxes are experimental on Modal: no names, GPUs, `env` or tags yet.
