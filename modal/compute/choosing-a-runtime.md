---
url: https://alchemy.run/modal/compute/choosing-a-runtime
title: "Choosing a runtime"
description: "Functions, Servers, program sandboxes and sandboxes created at run time on Modal, and when to use each."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Modal runs all your code in containers that start from an image, on CPUs or GPUs you choose. Its runtimes differ in what starts a container and what stops it: Modal itself, scaling with calls or requests, or your code, creating a sandbox and terminating it.

## Modal manages less as you decide more

Functionper call

Serverper request

programSandbox + main

sandboxper task

your code

you

you

you

you

image

Modal

Modal

Modal

you

scaling

Modal

Modal

you

you

container lifetime

Modal

Modal

you

you

hardware

Modal

Modal

Modal

Modal

From left to right, Modal manages less and you decide more. Functions and Servers scale on their own; your code creates each sandbox and decides when it stops.

## The same program on every runtime

A Function, a Server and a program Sandbox are each a class that holds its code. The program inside is the same shape, an Effect that returns `fetch` and any methods you want to call:

```typescript
Effect.gen(function* () {
  return {
    fetch: Effect.succeed(HttpServerResponse.text("ok")),
    embed: Effect.fn(function* (text: string) {
      return { tokens: text.split(/\s+/).length };
    }),
  };
});
```

Alchemy bundles `main` into an image, and Modal builds it at deploy.

## Functions: scale with calls

A `Modal.Function` runs when it’s called. Modal starts containers as calls arrive, runs up to `maxConcurrentInputs` in each, and stops them when they’re idle. Callers use its methods by name, a schedule calls it on a cron, or `web: true` serves its `fetch` at a `*.modal.run` URL.

```typescript
export default class Embed extends Modal.Function<Embed>()(
  "Embed",
  { main: import.meta.url, app: Inference, gpu: "L4" },
  program,
) {}
```

Choose it for inference, batch jobs and anything you fan out. See [Functions](../services/functions.md).

<svg viewBox="0 0 677.4 102" style="max-width: 779px" role="img" aria-label="Each container takes one call at a time here. A second container starts when calls pile up, and both stop once idle."><g><text fill="currentColor" x="4" y="34">container 1</text> <g><rect stroke="currentColor" fill="none" x="116.60000000000001" y="18" width="56.000000000000014" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="172.60000000000002" y="18" width="123.19999999999993" height="24" rx="5"></rect><text fill="currentColor" x="234.2" y="34" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="295.79999999999995" y="18" width="123.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="357.4" y="34" text-anchor="middle">embed</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="677.4" y2="52"></line><text fill="currentColor" x="4" y="78">container 2</text> <g><rect stroke="currentColor" fill="none" x="239.8" y="62" width="55.99999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="295.79999999999995" y="62" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="362.99999999999994" y="78" text-anchor="middle">embed</text></g></g></svg> <svg viewBox="0 0 330 122" style="max-width: 379px" role="img" aria-label="Each container takes one call at a time here. A second container starts when calls pile up, and both stop once idle."><g><text fill="currentColor" x="0" y="22">container 1</text> <g><rect stroke="currentColor" fill="none" x="6.36" y="32" width="31.799999999999997" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="38.16" y="32" width="69.96" height="24" rx="5"></rect><text fill="currentColor" x="73.13999999999999" y="48" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="32" width="69.96" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="48" text-anchor="middle">embed</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">container 2</text> <g><rect stroke="currentColor" fill="none" x="76.32" y="86" width="31.799999999999997" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="86" width="76.32000000000001" height="24" rx="5"></rect><text fill="currentColor" x="146.28" y="102" text-anchor="middle">embed</text></g></g></svg>

callcold start

Each container takes one call at a time here. A second container starts when calls pile up, and both stop once idle.

## Servers: scale with concurrent requests

A `Modal.Server` is an HTTP server behind one routing region. Each container serves many requests at once, and Modal adds a container when they pass `targetConcurrency`.

```typescript
export default class Api extends Modal.Server<Api>()(
  "Api",
  { main: import.meta.url, app: Backend, targetConcurrency: 20 },
  program,
) {}
```

Choose it for APIs, streaming responses and WebSockets. See [Servers](../services/servers.md).

<svg viewBox="0 0 677.4 118" style="max-width: 779px" role="img" aria-label="One container serves requests side by side. Modal scales on how many are in flight, not on a queue of calls."><g><rect stroke="currentColor" fill="none" x="97.4" y="11" width="576" height="98" rx="8"></rect><text fill="currentColor" x="4" y="64">container 1</text> <g><rect stroke="currentColor" fill="none" x="127.80000000000001" y="18" width="201.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="228.6" y="34" text-anchor="middle">GET /</text></g> <g><rect stroke="currentColor" fill="none" x="161.4" y="48" width="291.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="307" y="64" text-anchor="middle">GET /stream</text></g> <g><rect stroke="currentColor" fill="none" x="251" y="78" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="318.2" y="94" text-anchor="middle">POST /embed</text></g> <g><rect stroke="currentColor" fill="none" x="463.80000000000007" y="18" width="145.5999999999999" height="24" rx="5"></rect><text fill="currentColor" x="536.6" y="34" text-anchor="middle">GET /</text></g></g></svg> <svg viewBox="0 0 330 128" style="max-width: 379px" role="img" aria-label="One container serves requests side by side. Modal scales on how many are in flight, not on a queue of calls."><g><rect stroke="currentColor" fill="none" x="0" y="29" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="22">container 1</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="32" width="114.48" height="24" rx="5"></rect><text fill="currentColor" x="69.96000000000001" y="48" text-anchor="middle">GET /</text></g> <g><rect stroke="currentColor" fill="none" x="31.8" y="62" width="165.35999999999999" height="24" rx="5"></rect><text fill="currentColor" x="114.47999999999999" y="78" text-anchor="middle">GET /stream</text></g> <g><rect stroke="currentColor" fill="none" x="82.68" y="92" width="76.32" height="24" rx="5"></rect><text fill="currentColor" x="165" y="108" text-anchor="start">POST /embed</text></g> <g><rect stroke="currentColor" fill="none" x="203.52" y="32" width="82.67999999999998" height="24" rx="5"></rect><text fill="currentColor" x="244.86" y="48" text-anchor="middle">GET /</text></g></g></svg>

One container serves requests side by side. Modal scales on how many are in flight, not on a queue of calls.

## Program sandboxes: your program, one container per create

A `Modal.Sandbox` with a `main` bundles your program into an image. Your code starts one with `create`, reaches the program’s port through a tunnel, and terminates it, or lets its timeout stop it.

```typescript
const sandboxes = yield* Modal.CreateSandbox(MySandbox);
const sandbox = yield* sandboxes.create({ timeout: "10 minutes" });
const port = yield* sandbox.getTcpPort();
```

Choose it when each user or session needs a copy of your own server. See [Programs](../sandboxes/programs.md).

<svg viewBox="0 0 662.6 124" style="max-width: 762px" role="img" aria-label="Each create starts one container running your program. It runs until your code terminates it or its timeout runs out."><line stroke-opacity="0.2" stroke="currentColor" x1="426.6" y1="24" x2="426.6" y2="118"></line><g><text fill="currentColor" x="4" y="56">agent-sam</text> <g><rect stroke="currentColor" fill="none" x="113.00000000000001" y="40" width="313.6" height="24" rx="5"></rect><text fill="currentColor" x="269.8" y="56" text-anchor="middle">serving</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="662.6" y2="74"></line><text fill="currentColor" x="4" y="100">agent-ana</text> <g><rect stroke="currentColor" fill="none" x="258.6" y="84" width="369.6" height="24" rx="5"></rect><text fill="currentColor" x="443.40000000000003" y="100" text-anchor="middle">serving</text></g></g> <text fill="currentColor" x="426.6" y="18" text-anchor="middle">terminate</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="Each create starts one container running your program. It runs until your code terminates it or its timeout runs out."><line stroke-opacity="0.2" stroke="currentColor" x1="190.79999999999998" y1="24" x2="190.79999999999998" y2="138"></line><g><text fill="currentColor" x="0" y="44">agent-sam</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="54" width="178.07999999999998" height="24" rx="5"></rect><text fill="currentColor" x="101.75999999999999" y="70" text-anchor="middle">serving</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">agent-ana</text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="108" width="209.88" height="24" rx="5"></rect><text fill="currentColor" x="200.33999999999997" y="124" text-anchor="middle">serving</text></g></g> <text fill="currentColor" x="190.79999999999998" y="18" text-anchor="middle">terminate</text></svg>

Each create starts one container running your program. It runs until your code terminates it or its timeout runs out.

## Sandboxes created at run time: one per task

`Modal.CreateSandbox` bound to an image gives a Worker or Lambda a client that creates sandboxes for code you don’t trust. Your code runs commands in each one, reads and writes its files, and terminates it when the task is done. `idleTimeout` stops one nobody is using.

```typescript
const sandboxes = yield* Modal.CreateSandbox(Warm);
const sandbox = yield* sandboxes.create({ idleTimeout: "10 minutes" });
yield* sandbox.exec(["pnpm", "test"], { cwd: "/work" });
```

Choose it for agent loops, code interpreters and test runs. See [Sandboxes](../sandboxes/sandboxes.md).

<svg viewBox="0 0 640.4 124" style="max-width: 736px" role="img" aria-label="Each task gets its own sandbox. Its files and processes last between commands, until your code terminates it or the idle timeout does."><line stroke-opacity="0.2" stroke="currentColor" x1="516.4" y1="24" x2="516.4" y2="118"></line><g><text fill="currentColor" x="4" y="56">task-1</text> <g><rect stroke="currentColor" fill="none" x="79.60000000000001" y="40" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="146.8" y="56" text-anchor="middle">pnpm test</text></g> <g><rect stroke="currentColor" fill="none" x="247.60000000000002" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="298" y="56" text-anchor="middle">pnpm lint</text></g> <g><rect stroke="currentColor" fill="none" x="348.4" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="432.4" y="56" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="640.4" y2="74"></line><text fill="currentColor" x="4" y="100">task-2</text> <g><rect stroke="currentColor" fill="none" x="158" y="84" width="268.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="292.40000000000003" y="100" text-anchor="middle">agent loop</text></g></g> <text fill="currentColor" x="516.4" y="18" text-anchor="middle">idle timeout</text></svg> <svg viewBox="0 0 330 159" style="max-width: 379px" role="img" aria-label="Each task gets its own sandbox. Its files and processes last between commands, until your code terminates it or the idle timeout does."><line stroke-opacity="0.2" stroke="currentColor" x1="254.4" y1="24" x2="254.4" y2="153"></line><g><text fill="currentColor" x="0" y="44">task-1</text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="76.32000000000001" height="24" rx="5"></rect><text fill="currentColor" x="44.52" y="70" text-anchor="middle">pnpm test</text></g> <g><rect stroke="currentColor" fill="none" x="101.76" y="54" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="101.76" y="90" text-anchor="start">pnpm lint</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="206.7" y="70" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">task-2</text> <g><rect stroke="currentColor" fill="none" x="50.88" y="123" width="152.64000000000001" height="24" rx="5"></rect><text fill="currentColor" x="127.20000000000002" y="139" text-anchor="middle">agent loop</text></g></g> <text fill="currentColor" x="254.4" y="18" text-anchor="middle">idle timeout</text></svg>

commandidle, still billed

Each task gets its own sandbox. Its files and processes last between commands, until your code terminates it or the idle timeout does.

A [persistent sandbox](../sandboxes/persistent.md) keeps its files between sessions instead, and resumes where the last one stopped.

## Mix runtimes in one app

A Worker creates a sandbox per user and calls a GPU Function, and both read model weights from one Volume:

ApiWorker

MyEnvenvironment

session-samSandbox

EmbedFunction · L4

WeightsVolume

The Worker, Api, runs outside Modal and reaches both runtimes through Modal's API. The sandbox and the Function mount the same Volume in their environment.
