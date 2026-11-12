---
url: https://alchemy.run/daytona/sandboxes/sandboxes
title: "Sandboxes"
description: "How a Daytona sandbox moves between started, stopped, archived and deleted, and how to create sandboxes per request or per user from a Worker, or declare a long-lived dev box in your Stack."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **sandbox** is an isolated Linux machine: its own filesystem, processes and network, booted from a snapshot. Its disk survives a stop, so Daytona stops a sandbox nobody is using, and starts it again later where it left off. You pay for CPU and memory only while it runs.

## A sandbox stops when idle and starts where it left off

<svg viewBox="-4 -55.014495755427525 776 283.0144957554275" style="max-width: 892px" role="img" aria-label="autoStopInterval (15 minutes by default) stops an idle sandbox, autoArchiveInterval (7 days) archives a stopped one, and autoDeleteInterval, when you set it, deletes a stopped one. Starting a stopped or archived sandbox brings its files back."><defs><marker id="alc-sm-arrow-wflfff" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 200 63 Q 260 63 316 63" marker-end="url(#alc-sm-arrow-wflfff)"></path><path stroke="currentColor" fill="none" d="M 380.8333333333333 86 Q 260 231 141.72740426532457 89.07288511838951" marker-end="url(#alc-sm-arrow-wflfff)"></path><path stroke="currentColor" fill="none" d="M 480 63 Q 540 63 596 63" marker-end="url(#alc-sm-arrow-wflfff)"></path><path stroke="currentColor" fill="none" d="M 641.6666666666666 40 Q 400 -105 161.7633050361835 37.94201697828989" marker-end="url(#alc-sm-arrow-wflfff)"></path><path stroke="currentColor" fill="none" d="M 449.53846153846155 86 Q 540 128 626.8335025905448 168.3155547741815" marker-end="url(#alc-sm-arrow-wflfff)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-wflfff)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">started</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">billed while it runs</text></g> <g><rect stroke="currentColor" fill="none" x="320" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="400" y="61" text-anchor="middle">stopped</text> <text fill="currentColor" x="400" y="76" text-anchor="middle">disk kept</text></g> <g><rect stroke="currentColor" fill="none" x="600" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="680" y="61" text-anchor="middle">archived</text> <text fill="currentColor" x="680" y="76" text-anchor="middle">slower to start</text></g> <g><rect stroke="currentColor" fill="none" x="600" y="170" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="604" y="174" width="152" height="38" rx="19"></rect><text fill="currentColor" x="680" y="197" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="201.2" y="53" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="259" y="66" text-anchor="middle">autoStopInterval</text></g> <g><rect stroke="currentColor" fill="none" x="239.14018439966446" y="149.26822127959738" width="43" height="17" rx="4"></rect><text fill="currentColor" x="260.64018439966446" y="162.26822127959738" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="471.3" y="53" width="135.39999999999998" height="17" rx="4"></rect><text fill="currentColor" x="539" y="66" text-anchor="middle">autoArchiveInterval</text></g> <g><rect stroke="currentColor" fill="none" x="379.3574929257125" y="-43.014495755427525" width="43" height="17" rx="4"></rect><text fill="currentColor" x="400.8574929257125" y="-30.014495755427525" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="474.6929910322516" y="117.57888869354537" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="539.0929910322516" y="130.57888869354537" text-anchor="middle">autoDeleteInterval</text></g></svg> <svg viewBox="-12.972531828189709 -4 424.9725318281897 378" style="max-width: 489px" role="img" aria-label="autoStopInterval (15 minutes by default) stops an idle sandbox, autoArchiveInterval (7 days) archives a stopped one, and autoDeleteInterval, when you set it, deletes a stopped one. Starting a stopped or archived sandbox brings its files back."><defs><marker id="alc-sm-arrow-n-wflfff" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 120 86 Q 120 132 120 174" marker-end="url(#alc-sm-arrow-n-wflfff)"></path><path stroke="currentColor" fill="none" d="M 164.16 178 Q 252.48 132 167.70765633666792 87.84773767534787" marker-end="url(#alc-sm-arrow-n-wflfff)"></path><path stroke="currentColor" fill="none" d="M 120 224 Q 120 270 120 312" marker-end="url(#alc-sm-arrow-n-wflfff)"></path><path stroke="currentColor" fill="none" d="M 97.92 316 Q -12.47999999999999 201 95.14987268724114 88.88554928412381" marker-end="url(#alc-sm-arrow-n-wflfff)"></path><path stroke="currentColor" fill="none" d="M 153.33333333333331 224 Q 220 270 283.37434620441667 313.7282988810475" marker-end="url(#alc-sm-arrow-n-wflfff)"></path><g><g><circle stroke="currentColor" fill="none" cx="120" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="120" y1="20" x2="120" y2="36" marker-end="url(#alc-sm-arrow-n-wflfff)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">started</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">billed while it runs</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="199" text-anchor="middle">stopped</text> <text fill="currentColor" x="120" y="214" text-anchor="middle">disk kept</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="337" text-anchor="middle">archived</text> <text fill="currentColor" x="120" y="352" text-anchor="middle">slower to start</text></g> <g><rect stroke="currentColor" fill="none" x="240" y="316" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="244" y="320" width="152" height="38" rx="19"></rect><text fill="currentColor" x="320" y="343" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="62.2" y="121" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="120" y="134" text-anchor="middle">autoStopInterval</text></g> <g><rect stroke="currentColor" fill="none" x="209.20691408416698" y="122.46193441883696" width="43" height="17" rx="4"></rect><text fill="currentColor" x="214.20691408416698" y="135.46193441883696" text-anchor="start">start</text></g> <g><rect stroke="currentColor" fill="none" x="52.300000000000004" y="259" width="135.39999999999998" height="17" rx="4"></rect><text fill="currentColor" x="120" y="272" text-anchor="middle">autoArchiveInterval</text></g> <g><rect stroke="currentColor" fill="none" x="-0.9725318281897088" y="191.72138732103096" width="43" height="17" rx="4"></rect><text fill="currentColor" x="37.02746817181029" y="204.72138732103096" text-anchor="end">start</text></g> <g><rect stroke="currentColor" fill="none" x="154.7769198844375" y="259.4320747202619" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="219.1769198844375" y="272.4320747202619" text-anchor="middle">autoDeleteInterval</text></g></svg>

autoStopInterval (15 minutes by default) stops an idle sandbox, autoArchiveInterval (7 days) archives a stopped one, and autoDeleteInterval, when you set it, deletes a stopped one. Starting a stopped or archived sandbox brings its files back.

Set `autoStopInterval: 0` to keep a sandbox running, and `ttl` to delete it a fixed time after it was created, whatever its state. VM sandboxes (`linux-vm`, `windows`) can also pause, keeping their memory, with `autoPauseInterval`.

## Create sandboxes at runtime

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Daytona from "@alchemy.run/daytona";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Base } from "./Images.ts";

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const sandboxes = yield* Daytona.CreateSandbox(Base);
    return {
      fetch: Effect.gen(function* () {
        const sandbox = yield* sandboxes.create({ labels: { task: "lint" }, autoStopInterval: 10 });
        const lint = yield* sandbox.exec("npx eslint --version");
        yield* sandboxes.delete(sandbox.sandboxId);
        return yield* HttpServerResponse.json(lint);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(Daytona.CreateSandboxHttp)),
) {}
```

The Worker’s key can create and delete sandboxes, nothing else. `create` takes Daytona’s own create request and waits until the sandbox has started.

## Pick the snapshot per call

```typescript
const sandboxes = yield* Daytona.CreateSandbox();
const python = yield* sandboxes.create({ snapshot: "daytonaio/sandbox:0.11.0" });
```

Without a base, `create` uses the `snapshot` you pass, or Daytona’s default.

## Find a sandbox by name or label

```typescript
const mine = yield* sandboxes.get(\`dev-${user}\`);
const lints = yield* sandboxes.list({ labels: { task: "lint" } });
```

`get` returns `undefined` when there is no such sandbox. With a base, `list` only returns sandboxes created from it.

## Find what a Worker created

Objects created at run time are tagged with the stack, stage and binding, and are not deleted when the stack is destroyed. On a sandbox the tags are Daytona labels, `alchemy.stack`, `alchemy.stage` and `alchemy.id` (Daytona doesn’t allow `::` in a label key), next to your own labels. List them by those labels to clean them up:

```typescript
const leftover = yield* sandboxes.list({ labels: { "alchemy.stage": "pr-42" } });
yield* Effect.forEach(leftover, (s) => sandboxes.delete(s.id));
```

`daytona sandbox list` in the CLI shows the same labels.

## Declare a dev box

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";

export const DevBox = Daytona.Sandbox("DevBox", {
  snapshot: Base,
  autoStopInterval: "30 minutes",
  autoArchiveInterval: "7 days",
});
```

Deploy starts it again if Daytona stopped or archived it. Set `state: "stopped"` or `"archived"` to keep it asleep.

## Drive the dev box from a Worker

```typescript
const devBox = yield* Daytona.ConnectSandbox(DevBox);
yield* devBox.startSandbox();
const test = yield* devBox.exec("git pull && pnpm test", { cwd: "/home/daytona/repo" });
```

`ConnectSandbox` returns the same handle as `CreateSandbox`, for the sandbox you declared.

## A sandbox’s size comes from its snapshot

A sandbox gets its CPU, memory, disk and GPUs from its snapshot. Declare a second snapshot for a bigger sandbox:

```typescript
export const Big = Daytona.Snapshot("Big", { image: "node:22-bookworm-slim", cpu: 4, memory: 8, disk: 10 });
```

## Most settings change in place

Labels, network rules, previews, secrets, the TTL and the auto-stop, auto-pause, auto-archive and auto-delete intervals change in place. A new snapshot, user, environment, mount list, linked sandbox or region replaces the sandbox.

To run a program you write in a declared sandbox, see [Programs in a sandbox](programs.md).

## Fork a sandbox

```typescript
const fork = yield* devBox.forkSandbox({ name: \`pr-${number}\` });
const forks = yield* devBox.getSandboxForks();
```

A fork copies the sandbox’s disk into a new sandbox.
