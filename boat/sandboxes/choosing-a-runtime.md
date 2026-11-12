---
url: https://alchemy.run/boat/sandboxes/choosing-a-runtime
title: "Choosing a runtime"
description: "Program sandboxes, a declared dev box, and sandboxes created per task, and when to use each."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Every way of running code on Boat is a sandbox: a Linux machine that stops on a timer with its disk saved. They differ in who starts the work, and who decides when the machine comes and goes.

## Boat manages less as you decide more

Each runtime hands you more of the machine:

programSandbox + main

dev boxConnectSandbox

per taskCreateSandbox

your code

you

you

you

processes

Boat

you

you

start & stop

Boat

you

you

how many machines

Boat

Boat

you

From left to right, Boat manages less and you decide more. A program runs itself; a dev box is one machine you drive; per-task sandboxes are as many machines as your code creates.

## Programs: your server at a URL

A `Boat.Sandbox` with a `main` runs your Effect program with Node.js. Each deploy that changes the code restarts it in the same sandbox, and its port answers at `https://<subdomain>-3000.on.boat.dev`.

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends Boat.Sandbox<Api>()(
  "Api",
  { main: import.meta.url, public: true },
  Effect.gen(function* () {
    return { fetch: Effect.succeed(HttpServerResponse.text("hello")) };
  }),
) {}
```

A program sandbox never stops on its own. Choose it for an API or a service that needs a whole machine: Docker, a desktop, a long-lived process. See [Programs](programs.md).

<svg viewBox="0 0 618.2 80" style="max-width: 711px" role="img" aria-label="One sandbox serves every request, and keeps serving. A deploy that changes the code restarts the process in place."><line stroke-opacity="0.2" stroke="currentColor" x1="326.2" y1="24" x2="326.2" y2="74"></line><g><text fill="currentColor" x="4" y="56">Api</text> <g><rect stroke="currentColor" fill="none" x="46.2" y="40" width="280" height="24" rx="5"></rect><text fill="currentColor" x="186.2" y="56" text-anchor="middle">v1 serving</text></g> <g><rect stroke="currentColor" fill="none" x="326.2" y="40" width="33.599999999999966" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="359.79999999999995" y="40" width="246.4000000000001" height="24" rx="5"></rect><text fill="currentColor" x="483" y="56" text-anchor="middle">v2 serving</text></g></g> <text fill="currentColor" x="326.2" y="18" text-anchor="middle">deploy</text></svg> <svg viewBox="0 0 330 90" style="max-width: 379px" role="img" aria-label="One sandbox serves every request, and keeps serving. A deploy that changes the code restarts the process in place."><line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="24" x2="159" y2="84"></line><g><text fill="currentColor" x="0" y="44">Api</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="159" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="70" text-anchor="middle">v1 serving</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="19.079999999999984" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="178.07999999999998" y="54" width="139.92000000000002" height="24" rx="5"></rect><text fill="currentColor" x="248.04" y="70" text-anchor="middle">v2 serving</text></g></g> <text fill="currentColor" x="159" y="18" text-anchor="middle">deploy</text></svg>

restart

One sandbox serves every request, and keeps serving. A deploy that changes the code restarts the process in place.

## A dev box: one machine you drive

A `Boat.Sandbox` without `main` is a machine the Stack keeps. A Worker reaches it with `ConnectSandbox`, runs commands in it, and resumes it when it has stopped.

```typescript
const devBox = yield* Boat.ConnectSandbox(DevBox);
yield* devBox.resume({ autoStop: "30 minutes" });
yield* devBox.exec("git pull && npm test", { cwd: "/home/user/app" });
```

It stops after its `autoStop` with its disk saved, so you pay for it only while it works. Choose it for an agent’s home, a preview environment, or a build box. See [Sandboxes](sandboxes.md#drive-a-declared-sandbox-from-a-worker).

<svg viewBox="0 0 640.4 80" style="max-width: 736px" role="img" aria-label="The dev box stops when its timer runs out and keeps its files. The next request resumes it."><line stroke-opacity="0.2" stroke="currentColor" x1="236.4" y1="24" x2="236.4" y2="74"></line><line stroke-opacity="0.2" stroke="currentColor" x1="432.4" y1="24" x2="432.4" y2="74"></line><g><text fill="currentColor" x="4" y="56">DevBox</text> <g><rect stroke="currentColor" fill="none" x="68.4" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="152.4" y="56" text-anchor="middle">npm test</text></g> <g><rect stroke="currentColor" fill="none" x="236.4" y="40" width="195.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="334.4" y="56" text-anchor="middle">stopped</text></g> <g><rect stroke="currentColor" fill="none" x="432.4" y="40" width="196" height="24" rx="5"></rect><text fill="currentColor" x="530.4" y="56" text-anchor="middle">npm test</text></g></g> <text fill="currentColor" x="236.4" y="18" text-anchor="middle">autoStop</text> <text fill="currentColor" x="432.4" y="18" text-anchor="middle">resume()</text></svg> <svg viewBox="0 0 330 90" style="max-width: 379px" role="img" aria-label="The dev box stops when its timer runs out and keeps its files. The next request resumes it."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="84"></line><line stroke-opacity="0.2" stroke="currentColor" x1="206.70000000000002" y1="24" x2="206.70000000000002" y2="84"></line><g><text fill="currentColor" x="0" y="44">DevBox</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="70" text-anchor="middle">npm test</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="54" width="111.30000000000003" height="24" rx="5"></rect><text fill="currentColor" x="151.05" y="70" text-anchor="middle">stopped</text></g> <g><rect stroke="currentColor" fill="none" x="206.70000000000002" y="54" width="111.29999999999998" height="24" rx="5"></rect><text fill="currentColor" x="262.35" y="70" text-anchor="middle">npm test</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">autoStop</text> <text fill="currentColor" x="206.70000000000002" y="18" text-anchor="middle">resume()</text></svg>

stopped, disk saved

The dev box stops when its timer runs out and keeps its files. The next request resumes it.

## Sandboxes per task: as many machines as you need

`CreateSandbox` gives a Worker or Function a client that creates sandboxes, from a snapshot when you bind one. Your code drives each one and deletes it when the task is done.

```typescript
const sandboxes = yield* Boat.CreateSandbox(Golden);
const sandbox = yield* sandboxes.create({ type: "small", autoStop: "10 minutes" });
yield* sandbox.exec("npm test");
yield* sandbox.delete();
```

Choose it for agent loops, code a user sends, and anything that needs a fresh machine each time. See [Sandboxes](sandboxes.md#create-sandboxes-while-your-app-runs).

<svg viewBox="0 0 640.4 146" style="max-width: 736px" role="img" aria-label="Each task gets a sandbox of its own, started from the same snapshot, and deleted when the task ends."><g><text fill="currentColor" x="4" y="34">task-1</text> <g><rect stroke="currentColor" fill="none" x="90.80000000000001" y="18" width="201.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="191.6" y="34" text-anchor="middle">npm test</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="640.4" y2="52"></line><text fill="currentColor" x="4" y="78">task-2</text> <g><rect stroke="currentColor" fill="none" x="152.4" y="62" width="308" height="24" rx="5"></rect><text fill="currentColor" x="306.4" y="78" text-anchor="middle">agent loop</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="640.4" y2="96"></line><text fill="currentColor" x="4" y="122">task-3</text> <g><rect stroke="currentColor" fill="none" x="348.4" y="106" width="224" height="24" rx="5"></rect><text fill="currentColor" x="460.4" y="122" text-anchor="middle">npm test</text></g></g></svg> <svg viewBox="0 0 330 176" style="max-width: 379px" role="img" aria-label="Each task gets a sandbox of its own, started from the same snapshot, and deleted when the task ends."><g><text fill="currentColor" x="0" y="22">task-1</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="32" width="114.48" height="24" rx="5"></rect><text fill="currentColor" x="69.96000000000001" y="48" text-anchor="middle">npm test</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">task-2</text> <g><rect stroke="currentColor" fill="none" x="47.699999999999996" y="86" width="174.9" height="24" rx="5"></rect><text fill="currentColor" x="135.15" y="102" text-anchor="middle">agent loop</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">task-3</text> <g><rect stroke="currentColor" fill="none" x="159" y="140" width="127.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="222.6" y="156" text-anchor="middle">npm test</text></g></g></svg>

Each task gets a sandbox of its own, started from the same snapshot, and deleted when the task ends.

## Mix runtimes in one app

One account runs all three at once. A Worker creates a sandbox per task from a snapshot, drives a dev box, and calls a program’s API:

ApiWorker

acmeaccount

Goldensnapshot

task-42sandbox per task

DevBoxdev box

Searchprogram

A Worker, Api, outside Boat creates task sandboxes from Golden, runs commands in DevBox, and calls the Search program at its URL. Each binding gets its own key.
