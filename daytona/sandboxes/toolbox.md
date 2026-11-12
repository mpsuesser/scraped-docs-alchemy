---
url: https://alchemy.run/daytona/sandboxes/toolbox
title: "Toolbox"
description: "Run commands, edit files, use git and language servers, open previews and SSH into a sandbox."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

The **toolbox** is the API inside every sandbox. A daemon in the sandbox runs your commands, processes, terminals and file operations, and your calls reach it only through Daytona’s toolbox proxy, which checks your API key and passes each call into the sandbox. Ports are reached the same way, through preview URLs, and a shell through SSH access you create.

## Every call goes through the toolbox proxy

<svg viewBox="0 0 495.5999999999999 362" style="max-width: 570px" role="img" aria-label="Every toolbox call goes through Daytona's proxy with your key, and the daemon inside the sandbox runs it. A session it started keeps running between calls."><defs><marker id="alc-bd-arrow-c1zo6f" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="118" width="211.79999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="250" width="463.5999999999999" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="121.89999999999999" y1="64" x2="121.89999999999999" y2="144" marker-end="url(#alc-bd-arrow-c1zo6f)"></line><text fill="currentColor" x="131.89999999999998" y="110">exec</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="121.89999999999999" y1="196" x2="121.89999999999999" y2="276" marker-end="url(#alc-bd-arrow-c1zo6f)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="209.79999999999998" y1="304" x2="281.79999999999995" y2="304" marker-end="url(#alc-bd-arrow-c1zo6f)"></line><text fill="currentColor" x="247.79999999999995" y="296" text-anchor="middle">runs</text></g> <text fill="currentColor" x="215.79999999999998" y="137" text-anchor="end">Daytona</text> <text fill="currentColor" x="467.5999999999999" y="269" text-anchor="end">sandbox</text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="37" text-anchor="middle">your Worker</text> <text fill="currentColor" x="121.89999999999999" y="53" text-anchor="middle">API key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="148" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="169" text-anchor="middle">toolbox proxy</text> <text fill="currentColor" x="121.89999999999999" y="185" text-anchor="middle">checks the key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="280" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="301" text-anchor="middle">daemon</text> <text fill="currentColor" x="121.89999999999999" y="317" text-anchor="middle">exec · files · pty</text></g> <g><rect stroke="currentColor" fill="none" x="285.79999999999995" y="280" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="373.69999999999993" y="301" text-anchor="middle">session dev</text> <text fill="currentColor" x="373.69999999999993" y="317" text-anchor="middle">pnpm dev, still running</text></g></svg> <svg viewBox="0 0 243.79999999999998 542" style="max-width: 280px" role="img" aria-label="Every toolbox call goes through Daytona's proxy with your key, and the daemon inside the sandbox runs it. A session it started keeps running between calls."><defs><marker id="alc-bd-arrow-hyjxzn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="134" width="211.79999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="282" width="211.79999999999998" height="244" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="121.89999999999999" y1="64" x2="121.89999999999999" y2="160" marker-end="url(#alc-bd-arrow-hyjxzn)"></line><text fill="currentColor" x="131.89999999999998" y="118">exec</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="121.89999999999999" y1="212" x2="121.89999999999999" y2="308" marker-end="url(#alc-bd-arrow-hyjxzn)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="121.89999999999999" y1="360" x2="121.89999999999999" y2="456" marker-end="url(#alc-bd-arrow-hyjxzn)"></line><text fill="currentColor" x="131.89999999999998" y="414">runs</text></g> <text fill="currentColor" x="215.79999999999998" y="153" text-anchor="end">Daytona</text> <text fill="currentColor" x="215.79999999999998" y="301" text-anchor="end">sandbox</text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="37" text-anchor="middle">your Worker</text> <text fill="currentColor" x="121.89999999999999" y="53" text-anchor="middle">API key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="185" text-anchor="middle">toolbox proxy</text> <text fill="currentColor" x="121.89999999999999" y="201" text-anchor="middle">checks the key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="333" text-anchor="middle">daemon</text> <text fill="currentColor" x="121.89999999999999" y="349" text-anchor="middle">exec · files · pty</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="460" width="175.79999999999998" height="48" rx="8"></rect><text fill="currentColor" x="121.89999999999999" y="481" text-anchor="middle">session dev</text> <text fill="currentColor" x="121.89999999999999" y="497" text-anchor="middle">pnpm dev, still running</text></g></svg>

Every toolbox call goes through Daytona's proxy with your key, and the daemon inside the sandbox runs it. A session it started keeps running between calls.

Every sandbox handle carries the toolbox, with Daytona’s own parameters. Get a handle from `CreateSandbox` or `ConnectSandbox`.

## Run a shell command

```typescript
const result = yield* sandbox.exec("ls -la && cat package.json", { cwd: "/home/daytona/repo" });
// { exitCode: 0, result: "..." }
```

A non-zero exit comes back in `exitCode`; it doesn’t fail the Effect. Daytona combines stdout and stderr into `result`.

An array runs as an argument vector, with no shell to quote for:

```typescript
yield* sandbox.exec(["git", "commit", "-m", message], { cwd: "/home/daytona/repo" });
```

## Start a process

```typescript
const server = yield* sandbox.start(["python3", "-m", "http.server", "8080"], { cwd: "/srv" });
yield* server.output.pipe(
  Stream.runForEach((out) => Console.log(out.fd, new TextDecoder().decode(out.data))),
);
```

`start` runs the command in its own session and returns while it runs: `output` streams stdout and stderr, `write` types into its stdin, `wait` resolves with its exit code, and `kill()` ends it (Daytona takes no signal). Closing the scope only detaches.

## Keep a shell session

```typescript
yield* sandbox.process.createSession({ sessionId: "dev" });
yield* sandbox.process.sessionExecuteCommand({
  sessionId: "dev",
  command: "pnpm dev --port 3000",
  runAsync: true,
});
```

For a shell whose state carries from command to command, use a session. Sessions keep running between calls. `getSessionCommandLogs` returns the output so far, with `stdout` and `stderr` apart.

## Follow a command’s output

```typescript
yield* sandbox.process
  .followSessionCommandLogs({ sessionId: "dev", commandId: started.cmdId })
  .pipe(Stream.runForEach((piece) => Console.log(piece.stream, piece.text)));
```

The stream ends when the command exits. `followEntrypointLogs()` follows the sandbox’s entrypoint the same way.

## Open a terminal

```typescript
const pty = yield* sandbox.pty.open({ cols: 120, rows: 30 });
yield* pty.write("pnpm install && exit\n");
yield* pty.output.pipe(Stream.decodeText(), Stream.runForEach(Console.log));
const { exitCode } = yield* pty.exit;
```

The connection closes with its scope, and the terminal keeps running until it exits or you `kill` it. `pty.connect(sessionId)` attaches to a terminal that already runs. Daytona starts the user’s shell; type the command you want into it.

## Read and write files

```typescript
yield* sandbox.files.makeDir("/home/daytona/out");
yield* sandbox.files.write("/home/daytona/out/report.json", JSON.stringify(report));
const text = yield* sandbox.files.readText("/home/daytona/out/report.json");
const entries = yield* sandbox.files.list("/home/daytona/out");
```

`files` also has `read` (bytes), `stat` and `remove`. Daytona’s own file API is on `fs`, with search and replace:

```typescript
const hits = yield* sandbox.fs.findInFiles({ path: "/home/daytona/repo/src", pattern: "TODO" });
```

## Move many files at once

```typescript
yield* sandbox.fs.uploadFiles({
  files: [
    { path: "/home/daytona/repo/.env", content: "PORT=3000\n" },
    { path: "/home/daytona/repo/logo.png", content: pngBytes },
  ],
});
const { files, errors } = yield* sandbox.fs.downloadFiles({ paths: ["/home/daytona/out/a.json", "/home/daytona/out/b.json"] });
```

A path that can’t be read comes back in `errors`; the rest still download.

## Clone and commit with git

```typescript
yield* sandbox.git.cloneRepository({ url: "https://github.com/alchemy-run/alchemy", path: "/home/daytona/repo" });
const status = yield* sandbox.git.getStatus({ path: "/home/daytona/repo" });
```

## Open a port’s preview

```typescript
const port = yield* sandbox.getTcpPort(3000);
const res = yield* port.fetch(HttpClientRequest.get("/health"));
```

`getTcpPort` gives you a port by its preview URL, and its `fetch` sends a request there with the token, or proxies one your Worker received. Preview URLs, signed URLs and `public` are in [Network and previews](../networking/network.md#reach-a-port-from-outside).

## Open an SSH session

```typescript
const ssh = yield* sandbox.createSshAccess({ expiresInMinutes: 60 });
```

## Run a language server or code interpreter

```typescript
yield* sandbox.lsp.start({ languageId: "typescript", pathToProject: "/home/daytona/repo" });
const ctx = yield* sandbox.interpreter.createInterpreterContext({ language: "python" });
const run = yield* sandbox.interpreter.executeInterpreterCode({
  contextId: ctx.id,
  code: "import pandas as pd\nprint(pd.__version__)",
  timeout: 30,
});
// { stdout: "2.2.3\n", stderr: "", error: undefined }
```

A context keeps its variables between runs. An exception the code raises comes back in `error`; a run past its `timeout` fails with `InterpreterTimeout`. `streamInterpreterCode` streams the same output as it is produced.

## Drive a desktop with computer use

```typescript
yield* sandbox.computerUse.startComputerUse();
const shot = yield* sandbox.computerUse.takeScreenshot();
```

Computer use needs a snapshot with a desktop. It is billed like any running sandbox.

## Read a sandbox’s telemetry

```typescript
const logs = yield* sandbox.getSandboxLogs({ from, to });
```

Point the whole organization’s telemetry at your own backend with [OrganizationOtelConfig](../observability/telemetry.md#export-telemetry-to-your-backend).
