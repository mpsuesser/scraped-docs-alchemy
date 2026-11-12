---
url: https://alchemy.run/archil/compute/choosing-a-runtime
title: "Choosing a runtime"
description: "Serverless exec, program sandboxes, sandboxes created at run time, and mounts on your own hosts, and when to use each."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Archil runs code next to a disk in four ways. Three of them are Firecracker microVMs that Archil runs, and the fourth is a host you already run, with the disk mounted. They differ in how long the code lives and who keeps it running.

## Archil manages less as you decide more

execper call

programsandbox + main

sandboxper task

mountyour host

your code

you

you

you

you

processes

Archil

Archil

you

you

machine lifetime

Archil

Archil

you

you

image

Archil

Archil

you

you

host & client

Archil

Archil

Archil

you

From left to right, Archil manages less and you decide more. A mount runs on a host you already run, so everything under your code is yours.

## Serverless exec: one call, one microVM

`Archil.ExecDisk` runs one command against a disk. Each call boots a microVM with the disk at `/mnt/archil`, runs the command to completion, and shuts down. Many calls run at once, and each one is a single charge.

```typescript
const dataset = yield* Archil.ExecDisk(Dataset);
const result = yield* dataset.exec("wc -l /mnt/archil/*.csv");
```

A call runs for at most five minutes and keeps nothing but what it writes to the disk. Choose it for an agent’s shell tool, a search, or a batch over a dataset. See [Disks](../storage/disks.md#run-commands-against-a-disk).

<svg viewBox="0 0 640.4 146" style="max-width: 736px" role="img" aria-label="Each call gets its own microVM with the disk mounted, and it's gone when the command exits."><g><text fill="currentColor" x="4" y="34">call 1</text> <g><rect stroke="currentColor" fill="none" x="68.4" y="18" width="33.599999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="102" y="18" width="145.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="174.8" y="34" text-anchor="middle">wc -l</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="640.4" y2="52"></line><text fill="currentColor" x="4" y="78">call 2</text> <g><rect stroke="currentColor" fill="none" x="124.4" y="62" width="33.599999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="158" y="62" width="257.6" height="24" rx="5"></rect><text fill="currentColor" x="286.8" y="78" text-anchor="middle">grep TODO</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="640.4" y2="96"></line><text fill="currentColor" x="4" y="122">call 3</text> <g><rect stroke="currentColor" fill="none" x="348.4" y="106" width="33.60000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="382" y="106" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="449.2" y="122" text-anchor="middle">git pull</text></g></g></svg> <svg viewBox="0 0 330 176" style="max-width: 379px" role="img" aria-label="Each call gets its own microVM with the disk mounted, and it's gone when the command exits."><g><text fill="currentColor" x="0" y="22">call 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="19.08" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="19.08" y="32" width="82.68" height="24" rx="5"></rect><text fill="currentColor" x="60.42" y="48" text-anchor="middle">wc -l</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">call 2</text> <g><rect stroke="currentColor" fill="none" x="31.8" y="86" width="19.080000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="50.88" y="86" width="146.28" height="24" rx="5"></rect><text fill="currentColor" x="124.02000000000001" y="102" text-anchor="middle">grep TODO</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">call 3</text> <g><rect stroke="currentColor" fill="none" x="159" y="140" width="19.079999999999984" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="140" width="76.32000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.24" y="156" text-anchor="middle">git pull</text></g></g></svg>

commandboot

Each call gets its own microVM with the disk mounted, and it's gone when the command exits.

## Program sandboxes: your program at a URL

An `Archil.Sandbox` with a `main` runs your program. Archil’s service manager keeps it running, and its port gets a public `https://` URL. The sandbox pauses when idle, and the next request wakes it.

```typescript
import * as Archil from "@alchemy.run/archil";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends Archil.Sandbox<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    return { fetch: Effect.succeed(HttpServerResponse.text("hello")) };
  }),
) {}
```

Choose it for APIs and websites that live next to their data. See [Programs](programs.md).

<svg viewBox="0 0 618.2 80" style="max-width: 711px" role="img" aria-label="One sandbox serves every request. It pauses once idle, and a request to its URL wakes it with its memory intact."><line stroke-opacity="0.2" stroke="currentColor" x1="404.6" y1="24" x2="404.6" y2="74"></line><g><text fill="currentColor" x="4" y="56">Api</text> <g><rect stroke="currentColor" fill="none" x="46.2" y="40" width="179.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="135.8" y="56" text-anchor="middle">serving</text></g> <g><rect stroke="currentColor" fill="none" x="225.40000000000003" y="40" width="179.2" height="24" rx="5"></rect><text fill="currentColor" x="315" y="56" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="404.6" y="40" width="33.599999999999966" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="438.2" y="40" width="168.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="522.2" y="56" text-anchor="middle">serving</text></g></g> <text fill="currentColor" x="404.6" y="18" text-anchor="middle">request</text></svg> <svg viewBox="0 0 330 90" style="max-width: 379px" role="img" aria-label="One sandbox serves every request. It pauses once idle, and a request to its URL wakes it with its memory intact."><line stroke-opacity="0.2" stroke="currentColor" x1="203.52" y1="24" x2="203.52" y2="84"></line><g><text fill="currentColor" x="0" y="44">Api</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="101.76" height="24" rx="5"></rect><text fill="currentColor" x="50.88" y="70" text-anchor="middle">serving</text></g> <g><rect stroke="currentColor" fill="none" x="101.76" y="54" width="101.76" height="24" rx="5"></rect><text fill="currentColor" x="152.64000000000001" y="70" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="203.52" y="54" width="19.079999999999984" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="222.6" y="54" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="70" text-anchor="middle">serving</text></g></g> <text fill="currentColor" x="203.52" y="18" text-anchor="middle">request</text></svg>

paused, not billedwake

One sandbox serves every request. It pauses once idle, and a request to its URL wakes it with its memory intact.

## Sandboxes created at run time: one per task

`Archil.CreateSandbox` gives a Worker or Lambda a client that creates sandboxes by name, from an image. Your code drives each one: it runs commands, starts processes, opens terminals and deletes it when the task is done.

```typescript
const sandboxes = yield* Archil.CreateSandbox(MyImage);
const sandbox = yield* sandboxes.create({ name: \`task-${id}\` });
yield* sandbox.exec("npm test");
```

Choose it for agent loops and preview environments, where each user or task needs a machine of its own. See [Sandboxes](sandboxes.md).

<svg viewBox="0 0 640.4 124" style="max-width: 736px" role="img" aria-label="Each task gets its own sandbox. It keeps its files and memory while paused, until your code deletes it."><line stroke-opacity="0.2" stroke="currentColor" x1="516.4" y1="24" x2="516.4" y2="118"></line><g><text fill="currentColor" x="4" y="56">task-1</text> <g><rect stroke="currentColor" fill="none" x="90.80000000000001" y="40" width="167.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="174.79999999999998" y="56" text-anchor="middle">npm test</text></g> <g><rect stroke="currentColor" fill="none" x="258.79999999999995" y="40" width="123.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="320.4" y="56" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="382" y="40" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="449.2" y="56" text-anchor="middle">npm test</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="640.4" y2="74"></line><text fill="currentColor" x="4" y="100">task-2</text> <g><rect stroke="currentColor" fill="none" x="180.4" y="84" width="246.40000000000006" height="24" rx="5"></rect><text fill="currentColor" x="303.6" y="100" text-anchor="middle">agent loop</text></g></g> <text fill="currentColor" x="516.4" y="18" text-anchor="middle">delete</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="Each task gets its own sandbox. It keeps its files and memory while paused, until your code deletes it."><line stroke-opacity="0.2" stroke="currentColor" x1="254.4" y1="24" x2="254.4" y2="138"></line><g><text fill="currentColor" x="0" y="44">task-1</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="60.419999999999995" y="70" text-anchor="middle">npm test</text></g> <g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="54" width="69.96" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="70" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="54" width="76.32000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.24" y="70" text-anchor="middle">npm test</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">task-2</text> <g><rect stroke="currentColor" fill="none" x="63.6" y="108" width="139.92000000000002" height="24" rx="5"></rect><text fill="currentColor" x="133.56" y="124" text-anchor="middle">agent loop</text></g></g> <text fill="currentColor" x="254.4" y="18" text-anchor="middle">delete</text></svg>

Each task gets its own sandbox. It keeps its files and memory while paused, until your code deletes it.

## Mounts: a disk on your own host

`Archil.MountDisk` on a Cloudflare Container or an ECS task on EC2 installs Archil’s FUSE client in the host’s image. The host mounts the disk while it handles a request, with a mount token for that disk only.

```typescript
const dataset = yield* Archil.MountDisk(Dataset, { path: "/data", mode: "read-only" });
const path = yield* dataset.mount();
```

Choose it when the code already runs somewhere else and needs the files. See [Mounting](../storage/mounting.md).

Indexeryour Containermount token: Dataset

your programEffect

archil clientFUSE

mount

/data

mode

read-only

The host runs your program and the Archil client side by side. Its token mounts one disk.

## Mix runtimes in one app

One disk serves all of them at once. A program serves the files, a Worker runs commands against them, and your Container reads them to build an index:

<svg viewBox="0 0 408 352" style="max-width: 469px" role="img" aria-label="Three runtimes on one disk at once. An exclusive mount locks the disk for one writer, so mount it read-only or conditional where several clients share it."><defs><marker id="alc-dag-arrow-xekcax" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 L 314 56 L 314 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xekcax)"></path></g><g><path d="M 164 176 L 238 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xekcax)"></path><text fill="currentColor" x="201" y="170" text-anchor="middle">mounts</text></g> <g><path d="M 164 296 L 314 296 L 314 214" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-xekcax)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">ExecDisk</text></g> <g transform="translate(24, 144)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">program sandbox</text></g> <g transform="translate(24, 264)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Indexer</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">MountDisk</text></g> <g transform="translate(244, 144)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Dataset</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">disk</text></g></svg> <svg viewBox="0 0 312 324" style="max-width: 359px" role="img" aria-label="Three runtimes on one disk at once. An exclusive mount locks the disk for one writer, so mount it read-only or conditional where several clients share it."><defs><marker id="alc-dag-arrow-hamaqq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 125 42 L 244.5 42 L 244.5 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-hamaqq)"></path></g><g><path d="M 125 162 L 181 162" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-hamaqq)"></path><text fill="currentColor" x="153" y="156" text-anchor="middle">mounts</text></g> <g><path d="M 125 282 L 244.5 282 L 244.5 200" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-hamaqq)"></path></g><g transform="translate(10, 10)"><rect stroke="currentColor" fill="none" width="115" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="57.5" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="57.5" y="46" text-anchor="middle" dominant-baseline="middle">ExecDisk</text></g> <g transform="translate(10, 130)"><rect stroke="currentColor" fill="none" width="115" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="57.5" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="57.5" y="46" text-anchor="middle" dominant-baseline="middle">program sandbox</text></g> <g transform="translate(10, 250)"><rect stroke="currentColor" fill="none" width="115" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="57.5" y="28" text-anchor="middle" dominant-baseline="middle">Indexer</text> <text fill="currentColor" x="57.5" y="46" text-anchor="middle" dominant-baseline="middle">MountDisk</text></g> <g transform="translate(187, 130)"><rect stroke="currentColor" fill="none" width="115" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="57.5" y="28" text-anchor="middle" dominant-baseline="middle">Dataset</text> <text fill="currentColor" x="57.5" y="46" text-anchor="middle" dominant-baseline="middle">disk</text></g></svg>

Three runtimes on one disk at once. An exclusive mount locks the disk for one writer, so mount it read-only or conditional where several clients share it.
