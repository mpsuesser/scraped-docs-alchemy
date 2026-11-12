---
url: https://alchemy.run/command/what-is-command
title: "How commands work"
description: "How Command puts local processes into a Stack — builds that produce assets, one-off commands run for their effects, dev servers — and how memoization decides when each runs again."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Not everything a deploy needs happens in a cloud. A frontend has to be built before it’s uploaded. Code generation has to run before a bundle that imports it. A dev server has to be running while you develop. Command puts those local processes into your Stack, as resources, so they run in the right order and only when they need to.

A build runs before the site that uploads its output. An exec runs before what depends on its effects. A dev server runs only under `alchemy dev`.

<svg viewBox="0 0 698.4 208" style="max-width: 803px" role="img" aria-label="Whatever uses a command's result waits for it, and independent commands run at the same time. A dev server only runs under alchemy dev."><g><text fill="currentColor" x="4" y="28">vite-build</text> <text fill="currentColor" x="4" y="42">Command.Build</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="18" width="280" height="24" rx="5"></rect><text fill="currentColor" x="266.4" y="34" text-anchor="middle">npm run build</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="698.4" y2="52"></line><text fill="currentColor" x="4" y="72">Site</text> <text fill="currentColor" x="4" y="86">uploads dist/</text> <g><rect stroke="currentColor" fill="none" x="406.4" y="62" width="168" height="24" rx="5"></rect><text fill="currentColor" x="490.4" y="78" text-anchor="middle">upload dist/</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="698.4" y2="96"></line><text fill="currentColor" x="4" y="116">codegen</text> <text fill="currentColor" x="4" y="130">Command.Exec</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="106" width="139.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="196.39999999999998" y="122" text-anchor="middle">generate</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="140" x2="698.4" y2="140"></line><text fill="currentColor" x="4" y="160">Api</text> <text fill="currentColor" x="4" y="174">imports the code</text> <g><rect stroke="currentColor" fill="none" x="266.4" y="150" width="224" height="24" rx="5"></rect><text fill="currentColor" x="378.4" y="166" text-anchor="middle">bundle and deploy</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="126.4" y1="184" x2="686.4" y2="184"></line><text fill="currentColor" x="126.4" y="200" text-anchor="start">alchemy deploy</text></svg> <svg viewBox="0 0 330 248" style="max-width: 379px" role="img" aria-label="Whatever uses a command's result waits for it, and independent commands run at the same time. A dev server only runs under alchemy dev."><g><text fill="currentColor" x="0" y="22">vite-build <tspan fill="currentColor" dx="8">Command.Build</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="159" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="48" text-anchor="middle">npm run build</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">Site <tspan fill="currentColor" dx="8">uploads dist/</tspan></text> <g><rect stroke="currentColor" fill="none" x="159" y="86" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="206.7" y="102" text-anchor="middle">upload dist/</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">codegen <tspan fill="currentColor" dx="8">Command.Exec</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="140" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="39.75" y="156" text-anchor="middle">generate</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="170" x2="330" y2="170"></line><text fill="currentColor" x="0" y="184">Api <tspan fill="currentColor" dx="8">imports the code</tspan></text> <g><rect stroke="currentColor" fill="none" x="79.5" y="194" width="127.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="143.10000000000002" y="210" text-anchor="middle">bundle and deploy</text></g></g> <line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="224" x2="318" y2="224"></line><text fill="currentColor" x="0" y="240" text-anchor="start">alchemy deploy</text></svg>

Whatever uses a command's result waits for it, and independent commands run at the same time. A dev server only runs under alchemy dev.

## Builds produce an asset

A **build** runs a command that writes an output directory, like `npm run build` writing `dist`. The directory is the build’s result, and other resources take it as an input:

```typescript
const build = yield* Command.Build("vite-build", { command: "npm run build", cwd: "./frontend", outdir: "dist" });
```

The output’s hash is recorded too, so whatever uploads it knows when the files actually changed.

## Execs run for their effects

An **exec** runs a command for what it does, not what it writes: generate code, seed a database, call a script. It succeeds when the command exits with code `0`.

```typescript
yield* Command.Exec("codegen", { command: "npm run codegen", cwd: "./packages/api" });
```

Destroying an exec doesn’t undo what it did.

## Dev servers run while you develop

A **dev server** is a long-running process, like `vite dev`, that runs for as long as `alchemy dev` does. Its `url` is read from the process’s output, and other resources can use it. During `alchemy deploy`, it does nothing.

See [Dev servers](dev-servers.md).

## Memoization: running only when needed

A build or exec runs again only when something it depends on changed. Its inputs, the files under its working directory, are hashed, along with its command and environment. Unchanged inputs mean no run, so a deploy that touched only the backend never rebuilds the frontend.

See [Memoization](memoization.md).

## How Command fits in a Stack

Commands are resources with no cloud and no credentials. `Command.providers()` is already part of `AWS.providers()` and `Cloudflare.providers()`, and the Website resources on every cloud use builds and dev servers underneath. Declare one yourself for any local step the deploy depends on.
