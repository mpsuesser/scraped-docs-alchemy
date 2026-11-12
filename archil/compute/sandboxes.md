---
url: https://alchemy.run/archil/compute/sandboxes
title: "Sandboxes"
description: "Firecracker microVMs with disks mounted, declared as a dev box or created per task."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A sandbox is a Firecracker microVM that stays around between calls. Its root filesystem is a disk of its own, so files and installed packages survive a stop. It can mount other disks, serve ports, and run for up to 24 hours per session. After 30 minutes without a connection it pauses, keeping its memory and processes, and the next connection resumes it.

## A sandbox pauses when idle and resumes on the next connection

<svg viewBox="-4 -31.207106781186546 416 259.20710678118655" style="max-width: 478px" role="img" aria-label="A paused sandbox keeps its memory and running processes. A stopped one keeps only its disks. Neither is billed for compute."><defs><marker id="alc-sm-arrow-h0ysh9" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 123 86 Q 220 183 314.1715728752538 88.82842712474618" marker-end="url(#alc-sm-arrow-h0ysh9)"></path><path stroke="currentColor" fill="none" d="M 317 40 Q 220 -57 125.82842712474618 37.17157287525381" marker-end="url(#alc-sm-arrow-h0ysh9)"></path><path stroke="currentColor" fill="none" d="M 77 86 Q 35 128 74.17157287525382 167.17157287525382" marker-end="url(#alc-sm-arrow-h0ysh9)"></path><path stroke="currentColor" fill="none" d="M 123 170 Q 165 128 125.82842712474618 88.82842712474618" marker-end="url(#alc-sm-arrow-h0ysh9)"></path><path stroke="currentColor" fill="none" d="M 160 193 Q 220 193 276 193" marker-end="url(#alc-sm-arrow-h0ysh9)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-h0ysh9)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="100" y="61" text-anchor="middle">running</text> <text fill="currentColor" x="100" y="76" text-anchor="middle">billed</text></g> <g><rect stroke="currentColor" fill="none" x="280" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="340" y="61" text-anchor="middle">paused</text> <text fill="currentColor" x="340" y="76" text-anchor="middle">memory kept</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="170" width="120" height="46" rx="23"></rect><text fill="currentColor" x="100" y="191" text-anchor="middle">stopped</text> <text fill="currentColor" x="100" y="206" text-anchor="middle">disk kept</text></g> <g><rect stroke="currentColor" fill="none" x="280" y="170" width="120" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="284" y="174" width="112" height="38" rx="19"></rect><text fill="currentColor" x="340" y="197" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="168.09289321881346" y="125.20710678118655" width="102.39999999999999" height="17" rx="4"></rect><text fill="currentColor" x="219.29289321881345" y="138.20710678118655" text-anchor="middle">idle · pause()</text></g> <g><rect stroke="currentColor" fill="none" x="156.30710678118655" y="-19.207106781186546" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="220.70710678118655" y="-6.207106781186548" text-anchor="middle">connect · resume()</text></g> <g><rect stroke="currentColor" fill="none" x="30.492893218813457" y="117.29289321881345" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="55.292893218813454" y="130.29289321881345" text-anchor="middle">stop()</text></g> <g><rect stroke="currentColor" fill="none" x="119.90710678118656" y="118.70710678118655" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="144.70710678118655" y="131.70710678118655" text-anchor="middle">boot()</text></g> <g><rect stroke="currentColor" fill="none" x="194.2" y="183" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="219" y="196" text-anchor="middle">delete</text></g></svg>

A paused sandbox keeps its memory and running processes. A stopped one keeps only its disks. Neither is billed for compute.

## Declare a long-lived dev box

Declare a sandbox in your Stack to keep it across deploys. Each `mounts` entry names a disk and a `path`:

```typescript
export const Workspace = Archil.Disk("Workspace", {});

export const DevBox = Archil.Sandbox("DevBox", {
  baseImage: "node:24-bookworm",
  mounts: [{ disk: Workspace, path: "/workspace" }],
  ports: [3000],
});
```

A mount change updates the same sandbox: it stops, then starts with the new mounts.

## Pick a mount mode

An entry takes Archil’s own options: `mode` (`exclusive`, `conditional`, or `read-only`), `subdirectory`, and `queueMs`:

```typescript
export const Box = Archil.Sandbox("Box", {
  mounts: [
    { disk: Workspace, path: "/workspace", mode: "conditional" },
    { disk: Dataset, path: "/data", mode: "read-only", subdirectory: "exports" },
  ],
});
```

`exclusive` (the default) lets one client write the whole disk. `conditional` lets several write at once. Branches and the `shared` mode belong to the FUSE client, so they work in [mounts on your own hosts](../storage/mounting.md), not in a sandbox.

## Create sandboxes at run time

Bind an image, then create a sandbox per task. A mount’s disk is a handle, such as one from `CreateDisk`:

```typescript
export const MyImage = Archil.Image("MyImage", { image: "node:24-bookworm" });

const sandboxes = yield* Archil.CreateSandbox(MyImage);
const sandbox = yield* sandboxes.create({
  name: "task-1",
  mounts: [{ disk: home, path: "/home/user" }],
  idleTtlSeconds: 300,
});
```

## Run commands

`exec` runs a command to completion. A non-zero exit comes back in `exitCode`:

```typescript
const result = yield* sandbox.exec("npm test", { cwd: "/home/user/app" });
const upper = yield* sandbox.exec(["tr", "a-z", "A-Z"], { stdin: "hello" });
```

## Start a background process

`start` returns a process whose output you can stream. Closing the scope detaches; the process keeps running, and `attach` picks it up again by id:

```typescript
const server = yield* sandbox.start("npm run dev", { cwd: "/home/user/app" });
yield* server.output.pipe(Stream.take(10), Stream.runDrain);
```

## Open a terminal

`pty.open` runs a shell on a pseudo-terminal:

```typescript
const pty = yield* sandbox.pty.open({ cols: 120, rows: 40 });
yield* pty.write("ls -la\n");
yield* pty.resize({ cols: 160, rows: 50 });
```

## Move files in and out

```typescript
yield* sandbox.files.write("/home/user/app/.env", "MODE=test");
const env = yield* sandbox.files.readText("/home/user/app/.env");
```

Archil’s process API moves whole files. List, remove or create directories with `exec`.

## Pause, resume or fork it

```typescript
yield* sandbox.pause();
yield* sandbox.resume();
const copy = yield* sandbox.fork({ name: "task-1-retry" });
```

A fork copies the sandbox’s memory and disk into a new one. Archil only forks a sandbox with mounts while it is stopped. `stop()` and `boot()` stop and start it.

## Serve a port

A declared sandbox’s `ports`, or `exposePort` on any sandbox, serves a port at its own `https://<port>-<id>.<region>.archil.app` hostname. `getTcpPort` returns that port, and its `fetch` sends a request to it:

```typescript
yield* sandbox.exposePort(3000);
const port = yield* sandbox.getTcpPort(3000);
const response = yield* port.fetch(HttpClientRequest.get("/health"));
```

Pass `fetch` the request a handler received, and it proxies the method, path, query, headers and body, then returns the response:

```typescript
return yield* port.fetch(yield* HttpServerRequest);
```

Anyone with the URL can call a public port. Give a browser a port token instead, which it sends in the `X-Archil-Token` header:

```typescript
const token = yield* sandbox.createPortToken(3000, "1h");
```

## Drive a declared sandbox from a Worker

`ConnectSandbox` returns the same handle for a sandbox in your Stack:

```typescript
const devbox = yield* Archil.ConnectSandbox(DevBox);
yield* devbox.exec("pnpm build", { cwd: "/workspace" });
```

## Boot an image

A sandbox boots `node:24-bookworm` unless you give it a `baseImage` (any public `linux/amd64` image) or an `image` from [`Archil.Image`](images.md), for private registries and Dockerfiles.

## Run your own program

Give a sandbox a `main` and it runs your program at a public URL. See [Programs](programs.md).
