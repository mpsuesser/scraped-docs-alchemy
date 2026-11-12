---
url: https://alchemy.run/modal/sandboxes/sandboxes
title: "Sandboxes"
description: "Create, find, run commands in, and stop Modal sandboxes from any runtime."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Modal **Sandbox** is a container your code creates while it runs, to execute code it doesn’t trust: an agent’s commands, a user’s script. It starts from an image, belongs to an App, and runs until something stops it.

## A sandbox runs until a timeout stops it

A sandbox stops at its `timeout` (five minutes unless you set it, at most 24 hours), when it’s been idle for its `idleTimeout`, or when your code terminates it. Stopping its App terminates it too:

<svg viewBox="0 0 677.4 168" style="max-width: 779px" role="img" aria-label="The request's sandbox ends when the request's scope terminates it. session-sam stops once it's been idle for its idleTimeout. The job runs until its timeout. Each is gone once it stops: keep files that must outlive it on a Volume, or snapshot it."><line stroke-opacity="0.2" stroke="currentColor" x1="497.4" y1="24" x2="497.4" y2="162"></line><line stroke-opacity="0.2" stroke="currentColor" x1="609.4" y1="24" x2="609.4" y2="162"></line><g><text fill="currentColor" x="4" y="50">request</text> <text fill="currentColor" x="4" y="64">terminate()</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="161.4" y="56" text-anchor="middle">exec</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="94">session-sam</text> <text fill="currentColor" x="4" y="108">idleTimeout</text> <g><rect stroke="currentColor" fill="none" x="161.4" y="84" width="111.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="217.39999999999998" y="100" text-anchor="middle">exec</text></g> <g><rect stroke="currentColor" fill="none" x="273.4" y="84" width="56" height="24" rx="5"></rect><text fill="currentColor" x="301.4" y="100" text-anchor="middle">exec</text></g> <g><rect stroke="currentColor" fill="none" x="329.4" y="84" width="168" height="24" rx="5"></rect><text fill="currentColor" x="413.4" y="100" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="677.4" y2="118"></line><text fill="currentColor" x="4" y="138">job</text> <text fill="currentColor" x="4" y="152">timeout</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="128" width="504" height="24" rx="5"></rect><text fill="currentColor" x="357.4" y="144" text-anchor="middle">pnpm test</text></g></g> <text fill="currentColor" x="497.4" y="18" text-anchor="middle">idleTimeout</text> <text fill="currentColor" x="609.4" y="18" text-anchor="middle">timeout</text></svg> <svg viewBox="0 0 330 227" style="max-width: 379px" role="img" aria-label="The request's sandbox ends when the request's scope terminates it. session-sam stops once it's been idle for its idleTimeout. The job runs until its timeout. Each is gone once it stops: keep files that must outlive it on a Volume, or snapshot it."><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="38" x2="222.6" y2="221"></line><line stroke-opacity="0.2" stroke="currentColor" x1="286.2" y1="38" x2="286.2" y2="221"></line><g><text fill="currentColor" x="0" y="58">request <tspan fill="currentColor" dx="8">terminate()</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="68" width="63.6" height="24" rx="5"></rect><text fill="currentColor" x="31.8" y="84" text-anchor="middle">exec</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="98" x2="330" y2="98"></line><text fill="currentColor" x="0" y="112">session-sam <tspan fill="currentColor" dx="8">idleTimeout</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="122" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="63.599999999999994" y="138" text-anchor="middle">exec</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="122" width="31.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="95.39999999999999" y="158" text-anchor="start">exec</text></g> <g><rect stroke="currentColor" fill="none" x="127.2" y="122" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="174.9" y="138" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="167" x2="330" y2="167"></line><text fill="currentColor" x="0" y="181">job <tspan fill="currentColor" dx="8">timeout</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="191" width="286.2" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="207" text-anchor="middle">pnpm test</text></g></g> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">idleTimeout</text> <text fill="currentColor" x="286.2" y="32" text-anchor="middle">timeout</text></svg>

The request's sandbox ends when the request's scope terminates it. session-sam stops once it's been idle for its idleTimeout. The job runs until its timeout. Each is gone once it stops: keep files that must outlive it on a Volume, or snapshot it.

## Bind sandboxes in a runtime

`Modal.CreateSandbox` takes an image (sandboxes boot from it by default) or an app:

```typescript
const sandboxes = yield* Modal.CreateSandbox(Warm);
```

Provide `Modal.CreateSandboxHttp`. It gives the Worker its own Modal service user, a `contributor` in the App’s environment.

## Create one per request

```typescript
const sandbox = yield* Effect.acquireRelease(
  sandboxes.create({ timeout: "10 minutes" }),
  (sandbox) => Effect.ignore(sandbox.terminate()),
);
```

The sandbox stops when the request’s scope closes.

## Keep one per session by name

```typescript
const sandbox =
  (yield* sandboxes.fromName(\`session-${user}\`)) ??
  (yield* sandboxes.create({ name: \`session-${user}\`, idleTimeout: "10 minutes" }));
```

A name is unique among an app’s running sandboxes. `idleTimeout` stops it once nobody uses it.

## Run a command

```typescript
const result = yield* sandbox.exec(["pnpm", "test"], { cwd: "/work" });
```

`result` has `exitCode`, `stdout` and `stderr`; a non-zero exit is returned, not failed. A string runs with `sh -c`:

```typescript
const result = yield* sandbox.exec("pnpm install && pnpm test", { cwd: "/work" });
```

## Follow a running command

```typescript
const process = yield* sandbox.start("pnpm dev", { cwd: "/work" });
yield* Stream.runForEach(process.output, (chunk) => Console.log(chunk.fd, chunk.data));
```

`output` streams as the command runs. `write` feeds its stdin, `kill` signals it, and `wait` returns its exit code. Closing the scope detaches; the command keeps running.

## Open a terminal

```typescript
const terminal = yield* sandbox.pty.open({ cols: 120, rows: 40 });
yield* terminal.write("ls -la\n");
```

Modal can’t resize a running terminal, so `resize` fails with `Modal.SandboxPtyResizeUnsupported`.

## Read and write files

```typescript
yield* sandbox.files.write("/tmp/task.json", JSON.stringify(task));
const report = yield* sandbox.files.readText("/tmp/report.json");
```

`files` also has `read` (bytes), `list`, `stat`, `remove` and `makeDir`.

## Call a port through its tunnel

```typescript
const sandbox = yield* sandboxes.create({ encryptedPorts: [3000] });
const port = yield* sandbox.getTcpPort(3000);
const response = yield* port.fetch(HttpClientRequest.get("/health"));
```

`port.url` is the tunnel’s public URL. A Worker in front of the sandbox can forward the request it received:

```typescript
return yield* port.fetch(yield* HttpServerRequest.HttpServerRequest);
```

`port.connect()` opens a TCP socket through the tunnel instead: TLS for `encryptedPorts`, plain TCP for `unencryptedPorts`.

For a browser that shouldn’t hold Modal credentials, use `sandbox.connectToken({ port: 3000 })`.

## Size it with CPUs, memory and GPUs

```typescript
const sandbox = yield* sandboxes.create({ cpu: 2, memory: 4096, gpu: "A10G" });
```

## Find what your app created

Objects created at run time are tagged with the stack, stage and binding (`alchemy::stack`, `alchemy::stage`, `alchemy::id`) and are not deleted when the stack is destroyed. List them by those tags to clean them up:

```typescript
const mine = yield* sandboxes.list({ tags: { "alchemy::stack": "my-app", "alchemy::stage": "prod" } });
```

Your own `tags` merge under the three. Stopping the App still terminates every sandbox in it.
