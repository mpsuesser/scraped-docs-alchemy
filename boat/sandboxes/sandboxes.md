---
url: https://alchemy.run/boat/sandboxes/sandboxes
title: "Sandboxes"
description: "Boat sandboxes — machine sizes, auto-stop, stop and resume, labels — declared as a long-lived dev box or created per task from a Worker or Function."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **sandbox** is a Linux machine: Ubuntu with Docker, Node.js, Bun and Python, and `sudo` that needs no password. It comes in four sizes: `small` (2 vCPUs, 4 GB), `default`, `large` and `xlarge`, the last on Boat’s $100 plan and up. A sandbox starts in under a second, runs for its `autoStop`, then stops and keeps its disk.

## A sandbox stops, keeps its disk, and resumes

Stopping saves the disk, and resuming brings the files back:

<svg viewBox="-4 -27.124695047554425 776 177.24939009510888" style="max-width: 892px" role="img" aria-label="A sandbox runs until its auto-stop timer or stop(), then Boat saves its disk and stops it. resume() starts it again with its files; after 14 days stopped, Boat deletes the saved disk."><defs><marker id="alc-sm-arrow-5kjzig" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 148.75 86 Q 260 175 368.12652476222786 88.4987801902177" marker-end="url(#alc-sm-arrow-5kjzig)"></path><path stroke="currentColor" fill="none" d="M 371.25 40 Q 260 -49 151.8734752377721 37.501219809782306" marker-end="url(#alc-sm-arrow-5kjzig)"></path><path stroke="currentColor" fill="none" d="M 480 63 Q 540 63 596 63" marker-end="url(#alc-sm-arrow-5kjzig)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-5kjzig)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">ready</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="320" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="400" y="61" text-anchor="middle">archived</text> <text fill="currentColor" x="400" y="76" text-anchor="middle">stopped · disk saved</text></g> <g><rect stroke="currentColor" fill="none" x="600" y="40" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="604" y="44" width="152" height="38" rx="19"></rect><text fill="currentColor" x="680" y="67" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="201.41913119055693" y="121.12469504755444" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="259.21913119055694" y="134.12469504755444" text-anchor="middle">autoStop, stop()</text></g> <g><rect stroke="currentColor" fill="none" x="229.380868809443" y="-15.124695047554424" width="62.8" height="17" rx="4"></rect><text fill="currentColor" x="260.780868809443" y="-2.1246950475544235" text-anchor="middle">resume()</text></g> <g><rect stroke="currentColor" fill="none" x="474.6" y="53" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="539" y="66" text-anchor="middle">delete, or 14 days</text></g></svg> <svg viewBox="-14.468024373724575 -4 321.7360487474491 378" style="max-width: 370px" role="img" aria-label="A sandbox runs until its auto-stop timer or stop(), then Boat saves its disk and stops it. resume() starts it again with its files; after 14 days stopped, Boat deletes the saved disk."><defs><marker id="alc-sm-arrow-n-5kjzig" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 149.44 86 Q 208.32000000000002 132 152.59209749489824 175.53742383211073" marker-end="url(#alc-sm-arrow-n-5kjzig)"></path><path stroke="currentColor" fill="none" d="M 90.56 178 Q 31.67999999999998 132 87.40790250510175 88.46257616788925" marker-end="url(#alc-sm-arrow-n-5kjzig)"></path><path stroke="currentColor" fill="none" d="M 120 224 Q 120 270 120 312" marker-end="url(#alc-sm-arrow-n-5kjzig)"></path><g><g><circle stroke="currentColor" fill="none" cx="120" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="120" y1="20" x2="120" y2="36" marker-end="url(#alc-sm-arrow-n-5kjzig)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">ready</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="199" text-anchor="middle">archived</text> <text fill="currentColor" x="120" y="214" text-anchor="middle">stopped · disk saved</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="44" y="320" width="152" height="38" rx="19"></rect><text fill="currentColor" x="120" y="343" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="179.66802437372456" y="121.3843559580277" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="184.66802437372456" y="134.3843559580277" text-anchor="start">autoStop, stop()</text></g> <g><rect stroke="currentColor" fill="none" x="-2.4680243737245746" y="122.6156440419723" width="62.8" height="17" rx="4"></rect><text fill="currentColor" x="55.33197562627542" y="135.6156440419723" text-anchor="end">resume()</text></g> <g><rect stroke="currentColor" fill="none" x="55.6" y="259" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="120" y="272" text-anchor="middle">delete, or 14 days</text></g></svg>

A sandbox runs until its auto-stop timer or stop(), then Boat saves its disk and stops it. resume() starts it again with its files; after 14 days stopped, Boat deletes the saved disk.

Processes don’t survive a stop: a resumed sandbox has your files, and nothing running.

## Declare a sandbox

A sandbox in the Stack is one that outlives requests: a dev box, an agent’s home, a preview environment.

```typescript
import * as Boat from "@alchemy.run/boat";

export const DevBox = Boat.Sandbox("DevBox", {
  type: "large",
  autoStop: "60 minutes",
  setupScript: "git clone https://github.com/alchemy-run/alchemy ~/alchemy",
});
```

`setupScript` runs once, in the background, after the sandbox starts; `setupStatus` says how it went. `autoStop: false` never stops it. Changing the name, subdomain or `autoStop` updates the sandbox in place. A new `type`, `env`, `environment`, `setupScript`, `labels`, or a new or rebuilt `snapshot`, replaces it, and the old sandbox’s disk goes with it.

## Keep the account’s secrets out of untrusted code

A sandbox gets your account’s default environment, with its variables, secret files and credentials, unless you say otherwise. For a sandbox that runs someone else’s code, pass `noEnv`:

```typescript
export const Untrusted = Boat.Sandbox("Untrusted", { noEnv: true, env: { TASK: "lint" } });
```

It gets only its own `env`, and it can’t act on your account or other sandboxes. See [Environments & secrets](../security/environments.md) for sandboxes that should get some secrets and not others.

## Create sandboxes while your app runs

For a sandbox per user or per task, bind `CreateSandbox` in a Worker or Function, and create sandboxes while it runs:

```typescript
const sandboxes = yield* Boat.CreateSandbox();
// in fetch:
const sandbox = yield* Effect.acquireRelease(
  sandboxes.create({ type: "small", noEnv: true, autoStop: "10 minutes" }),
  (sandbox) => Effect.ignore(sandbox.delete()),
);
```

Provide `Boat.CreateSandboxHttp`. `create` takes Boat’s own parameters (`type`, `env`, `environment`, `noEnv`, `setupScript`, `labels`, `from`), and `autoStop`.

## Find a sandbox again by its labels

Labels mark a sandbox when it’s created. Find it by them on the next request, and resume it with its files:

```typescript
const [existing] = yield* sandboxes.list({ labels: { user } });
const sandbox = existing
  ? yield* sandboxes.connect(existing.id)
  : yield* sandboxes.create({ type: "small", autoStop: "10 minutes", labels: { user } });
yield* sandbox.resume({ autoStop: "10 minutes" });
```

Sandboxes a binding creates also carry `alchemy.stack`, `alchemy.stage` and `alchemy.id`. They aren’t deleted when the Stack is destroyed, so list them by those labels to clean up.

## Run commands

`exec` runs to completion and returns `{ exitCode, stdout, stderr }`. A non-zero exit is returned, not failed. A string runs in a shell, an array runs its arguments as given:

```typescript
const test = yield* sandbox.exec("npm test", { cwd: "/home/user/app" });
const version = yield* sandbox.exec(["node", "--version"]);
const sorted = yield* sandbox.exec("sort", { stdin: "b\na\n", env: { LC_ALL: "C" } });
```

Boat stops a command after 30 seconds unless you pass a `timeout`, up to 10 minutes; one that runs past it fails with `ExecTimedOut`. For anything longer, or a server, start it in the background and read its output as it comes:

```typescript
const server = yield* sandbox.start("npm run dev", { cwd: "/home/user/app" });
yield* server.output.pipe(
  Stream.take(5),
  Stream.runForEach((chunk) => Effect.log(new TextDecoder().decode(chunk.data))),
);
```

A background command keeps running after the request ends; `sandbox.processes.attach(server.processId)` picks it up again.

## Read and write files

Paths are relative to `/home/user`. Boat reaches files under `/home/user` and `/tmp`:

```typescript
yield* sandbox.files.write("app/config.json", JSON.stringify(config));
const log = yield* sandbox.files.readText("/tmp/build.log");
const entries = yield* sandbox.files.list("app");
```

A missing path fails with `FileNotFound`. Writing creates the parent directories.

## Drive a declared sandbox from a Worker

Bind `ConnectSandbox` to reach a sandbox the Stack declares, from a Worker or Function:

```typescript
const devBox = yield* Boat.ConnectSandbox(DevBox);
// in fetch:
yield* devBox.resume({ autoStop: "30 minutes" });
const result = yield* devBox.exec("git pull && npm test", { cwd: "/home/user/alchemy" });
```

Provide `Boat.ConnectSandboxHttp`. A call into a stopped sandbox fails with `SandboxNotRunning`, so resume it first. Resuming one that’s running does nothing.

To run your own server in a sandbox, see [Programs](programs.md).
