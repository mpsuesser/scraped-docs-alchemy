---
url: https://alchemy.run/e2b/sandboxes
title: "Sandboxes"
description: "Create sandboxes from a Worker, run commands, read and write files, and manage their lifecycle."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A sandbox is one microVM, started from a template’s snapshot. It runs for its **timeout**, counted from when it starts or resumes. When the timeout runs out, E2B kills it, or with `autoPause` pauses it: its memory, files and running processes are kept, and a resume picks up where it stopped.

## A sandbox lives for its timeout

<svg viewBox="-4 -25.16896473162245 416 253.16896473162245" style="max-width: 478px" role="img" aria-label="A sandbox starts running. Its timeout kills it, or with autoPause pauses it with its memory until something resumes it. kill() ends it from either state."><defs><marker id="alc-sm-arrow-5r9qr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 160 81 Q 220 99 276.1686948591154 82.14939154226538" marker-end="url(#alc-sm-arrow-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 314.44444444444446 40 Q 220 -45 128.52873214054424 37.324141073510205" marker-end="url(#alc-sm-arrow-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 121.23076923076923 86 Q 160 128 196.0561103582985 167.06078622149005" marker-end="url(#alc-sm-arrow-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 318.7692307692308 86 Q 280 128 243.9438896417015 167.06078622149005" marker-end="url(#alc-sm-arrow-5r9qr2)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-5r9qr2)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="100" y="67" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="280" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="340" y="61" text-anchor="middle">paused</text> <text fill="currentColor" x="340" y="76" text-anchor="middle">memory kept</text></g> <g><rect stroke="currentColor" fill="none" x="160" y="170" width="120" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="164" y="174" width="112" height="38" rx="19"></rect><text fill="currentColor" x="220" y="197" text-anchor="middle">killed</text></g> <g><rect stroke="currentColor" fill="none" x="184.34217371477888" y="80.28734788556635" width="69.4" height="17" rx="4"></rect><text fill="currentColor" x="219.04217371477887" y="93.28734788556635" text-anchor="middle">autoPause</text></g> <g><rect stroke="currentColor" fill="none" x="195.94329414624718" y="-13.168964731622449" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="220.74329414624717" y="-0.16896473162244874" text-anchor="middle">resume</text></g> <g><rect stroke="currentColor" fill="none" x="101.52171989726692" y="117.26519655537251" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="159.32171989726692" y="130.2651965553725" text-anchor="middle">timeout · kill()</text></g> <g><rect stroke="currentColor" fill="none" x="255.87828010273307" y="117.26519655537251" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="280.6782801027331" y="130.2651965553725" text-anchor="middle">kill()</text></g></svg> <svg viewBox="28 -16.033964731622444 343.4 252.03396473162243" style="max-width: 395px" role="img" aria-label="A sandbox starts running. Its timeout kills it, or with autoPause pauses it with its memory until something resumes it. kill() ends it from either state."><defs><marker id="alc-sm-arrow-n-5r9qr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 160 81 Q 199.7 92.91 235.56869485911537 82.14939154226538" marker-end="url(#alc-sm-arrow-n-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 273.84444444444443 40 Q 199.7 -26.72999999999999 128.52873214054424 37.324141073510205" marker-end="url(#alc-sm-arrow-n-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 116.61666666666666 86 Q 149.85 132 180.74085644411005 174.75765485744424" marker-end="url(#alc-sm-arrow-n-5r9qr2)"></path><path stroke="currentColor" fill="none" d="M 282.7833333333333 86 Q 249.54999999999998 132 218.65914355588993 174.75765485744424" marker-end="url(#alc-sm-arrow-n-5r9qr2)"></path><g><g><circle stroke="currentColor" fill="none" cx="100" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="20" x2="100" y2="36" marker-end="url(#alc-sm-arrow-n-5r9qr2)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="100" y="67" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="239.39999999999998" y="40" width="120" height="46" rx="23"></rect><text fill="currentColor" x="299.4" y="61" text-anchor="middle">paused</text> <text fill="currentColor" x="299.4" y="76" text-anchor="middle">memory kept</text></g> <g><rect stroke="currentColor" fill="none" x="139.7" y="178" width="120" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="143.7" y="182" width="112" height="38" rx="19"></rect><text fill="currentColor" x="199.7" y="205" text-anchor="middle">killed</text></g> <g><rect stroke="currentColor" fill="none" x="129.34217371477882" y="77.24234788556635" width="69.4" height="17" rx="4"></rect><text fill="currentColor" x="193.74217371477883" y="90.24234788556635" text-anchor="end">autoPause</text></g> <g><rect stroke="currentColor" fill="none" x="200.44329414624715" y="-4.033964731622444" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="205.44329414624715" y="8.966035268377556" text-anchor="start">resume</text></g> <g><rect stroke="currentColor" fill="none" x="91.46438077769419" y="121.18941371436105" width="115.6" height="17" rx="4"></rect><text fill="currentColor" x="149.26438077769419" y="134.18941371436105" text-anchor="middle">timeout · kill()</text></g> <g><rect stroke="currentColor" fill="none" x="225.3356192223058" y="121.18941371436105" width="49.599999999999994" height="17" rx="4"></rect><text fill="currentColor" x="250.1356192223058" y="134.18941371436105" text-anchor="middle">kill()</text></g></svg>

A sandbox starts running. Its timeout kills it, or with autoPause pauses it with its memory until something resumes it. kill() ends it from either state.

## Bind sandboxes in a Worker

You create sandboxes while your app runs, with `E2B.CreateSandbox`. Pass a template or snapshot to create from it, or nothing and pass `templateID` per call.

```typescript
// in the Worker's construction
const sandboxes = yield* E2B.CreateSandbox(MyTemplate);
```

Provide `E2B.CreateSandboxHttp` on the Worker. The Worker gets its own E2B API key as a secret, deleted when the Worker is destroyed.

## Create a sandbox per request

```typescript
// in fetch
const sandbox = yield* Effect.acquireRelease(sandboxes.create({ timeout: 120 }), (sandbox) =>
  Effect.ignore(sandbox.kill()),
);
```

The sandbox is killed when the request’s scope closes. `timeout` is in seconds; E2B kills the sandbox when it runs out.

## Keep a sandbox per user

```typescript
const [existing] = yield* sandboxes.list({ metadata: { user }, state: ["running", "paused"] });
const sandbox = existing
  ? yield* sandboxes.connect(existing.sandboxID, { timeout: 600 })
  : yield* sandboxes.create({ metadata: { user }, autoPause: true, timeout: 600 });
```

With `autoPause`, E2B pauses the sandbox instead of killing it when the timeout runs out, memory included. `connect` resumes it.

## Declare a sandbox the Stack owns

```typescript
import * as E2B from "@alchemy.run/e2b";
import { MyTemplate } from "./MyTemplate.ts";

export const DevBox = E2B.Sandbox("DevBox", { template: MyTemplate, timeout: "10 minutes" });
```

`E2B.Sandbox` declares one sandbox that lives as long as the stack. It runs for `timeout` after it starts or resumes, then E2B pauses it with its memory and files. The next request to it resumes it. A rebuilt `MyTemplate` replaces it.

```typescript
// in the Worker's construction
const devBox = yield* E2B.ConnectSandbox(DevBox);
// in fetch
const result = yield* devBox.exec("git pull && npm test", { cwd: "/home/user/repo" });
```

`ConnectSandbox` returns the same handle as `create`, with `sandboxId` read at runtime. Provide `E2B.ConnectSandboxHttp` on the Worker.

With `autoResume: false`, calls into a paused sandbox fail with `SandboxNotRunning` until you call `devBox.resume()`.

## Run commands

```typescript
const result = yield* sandbox.exec("npm test", { cwd: "/home/user/app" });
// { stdout, stderr, exitCode, pid, error }
```

A non-zero exit code is returned, not failed. `exec(["node", "--version"])` runs an argument vector without a shell. `stdin` feeds the command, and `timeout` kills it and fails with `E2B.ExecTimedOut`.

## Stream a command’s output

```typescript
const build = yield* sandbox.start("npm run build");
yield* build.output.pipe(Stream.runForEach((chunk) => Effect.log(chunk.fd, chunk.data.length)));
const { exitCode } = yield* build.wait;
```

`start` returns a `Process` with its `output`, `write`, `kill` and `wait`. Closing the scope detaches; the process keeps running. Re-attach with `sandbox.processes.attach({ pid })`.

## Read and write files

```typescript
yield* sandbox.files.write("/home/user/app/config.json", JSON.stringify(config));
const log = yield* sandbox.files.readText("/home/user/app/out.log");
const entries = yield* sandbox.files.list("/home/user/app");
```

`files.watch(path)` streams changes under a directory.

## Open a terminal

```typescript
const pty = yield* sandbox.pty.open({ cols: 120, rows: 40 });
yield* pty.write("ls\n");
yield* pty.resize({ cols: 160, rows: 48 });
```

`pty.output` streams the terminal’s bytes. Closing the scope detaches.

## Call a port

```typescript
const port = yield* sandbox.getTcpPort(3000);
const res = yield* port.fetch(HttpClientRequest.get("/health"));
port.url; // https://3000-<sandboxId>.e2b.app
```

`port.fetch` also takes the request a Worker received, and proxies it to the port:

```typescript
const port = yield* sandbox.getTcpPort(3000);
return yield* port.fetch(yield* HttpServerRequest);
```

## Extend, pause or kill it

```typescript
yield* sandbox.setTimeout("30 minutes");
yield* sandbox.pause();
yield* sandbox.kill();
```

## Clean up what your app created

Objects created at run time are tagged with the stack, stage and binding (`alchemy::stack`, `alchemy::stage`, `alchemy::id`) and are not deleted when the stack is destroyed. List them by those tags to clean them up.

```typescript
const left = yield* sandboxes.list({
  metadata: { "alchemy::stack": "my-app", "alchemy::stage": "prod" },
  state: ["running", "paused"],
});
yield* Effect.forEach(left, (sandbox) => sandboxes.kill(sandbox.sandboxID));
```

A template can’t be deleted while sandboxes run on it, so kill them before you destroy the stack.
