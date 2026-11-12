---
url: https://alchemy.run/boat/sandboxes/programs
title: "Programs"
description: "Run an Effect HTTP program in its own Boat sandbox, served at a URL, restarted in place when its code changes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **program** is your own server, run in a sandbox the Stack declares. Alchemy bundles it, uploads it into the sandbox and starts it with Node.js 24, then serves its port at `https://<subdomain>-3000.on.boat.dev`.

## Give a sandbox a program

Give `Boat.Sandbox` a `main` and an implementation:

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends Boat.Sandbox<Api>()(
  "Api",
  { main: import.meta.url, type: "small", noEnv: true, public: true },
  Effect.gen(function* () {
    return {
      fetch: Effect.gen(function* () {
        return yield* HttpServerResponse.json({ ok: true });
      }),
    };
  }),
) {}
```

Add it to the Stack and return its URL:

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import Api from "./src/Api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: Boat.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url.as<string>() };
  }),
);
```

The program listens on `port` (3000 unless you say otherwise), and reads `PORT` from its environment.

## A deploy restarts the process, not the sandbox

A program’s sandbox stays the same across deploys; only the process changes.

<svg viewBox="0 0 699.6 102" style="max-width: 805px" role="img" aria-label="Each deploy that changes the code or the program's env uploads the new bundle and restarts the process. The sandbox, its disk and its URL stay the same."><g><text fill="currentColor" x="4" y="34">alchemy deploy</text> <g><rect stroke="currentColor" fill="none" x="127.60000000000001" y="18" width="70.00000000000001" height="24" rx="5"></rect><text fill="currentColor" x="162.60000000000002" y="34" text-anchor="middle">v1 upload</text></g> <g><rect stroke="currentColor" fill="none" x="407.6" y="18" width="70" height="24" rx="5"></rect><text fill="currentColor" x="442.6" y="34" text-anchor="middle">v2 upload</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="699.6" y2="52"></line><text fill="currentColor" x="4" y="72">node index.mjs</text> <text fill="currentColor" x="4" y="86">in sandbox Api</text> <g><rect stroke="currentColor" fill="none" x="197.60000000000002" y="62" width="210" height="24" rx="5"></rect><text fill="currentColor" x="302.6" y="78" text-anchor="middle">v1 serving</text></g> <g><rect stroke="currentColor" fill="none" x="477.6" y="62" width="210" height="24" rx="5"></rect><text fill="currentColor" x="582.6" y="78" text-anchor="middle">v2 serving</text></g></g></svg> <svg viewBox="0 0 330 122" style="max-width: 379px" role="img" aria-label="Each deploy that changes the code or the program's env uploads the new bundle and restarts the process. The sandbox, its disk and its URL stay the same."><g><text fill="currentColor" x="0" y="22">alchemy deploy</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="39.75" height="24" rx="5"></rect><text fill="currentColor" x="45.75" y="48" text-anchor="start">v1 upload</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="32" width="39.75" height="24" rx="5"></rect><text fill="currentColor" x="204.75" y="48" text-anchor="start">v2 upload</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">node index.mjs <tspan fill="currentColor" dx="8">in sandbox Api</tspan></text> <g><rect stroke="currentColor" fill="none" x="39.75" y="86" width="119.25" height="24" rx="5"></rect><text fill="currentColor" x="99.375" y="102" text-anchor="middle">v1 serving</text></g> <g><rect stroke="currentColor" fill="none" x="198.75" y="86" width="119.25" height="24" rx="5"></rect><text fill="currentColor" x="258.375" y="102" text-anchor="middle">v2 serving</text></g></g></svg>

Each deploy that changes the code or the program's env uploads the new bundle and restarts the process. The sandbox, its disk and its URL stay the same.

An unchanged deploy leaves the process alone. A program sandbox never stops on its own unless you set `autoStop`. If it stopped anyway, or rebooted, the next deploy resumes it and starts the program again.

`env` on a program sandbox is the program’s: changing it restarts the process, without replacing the sandbox.

## Its URL needs a token unless it’s public

A program’s URL needs its token unless you set `public: true`. The token is `urlToken`, sent as the `_token` query parameter or the `_port_auth` cookie:

```typescript
const api = yield* Api;
return { url: api.url, token: api.urlToken };
```

See [Ports & URLs](ports.md).

## Use bindings from the program

A program uses bindings like any other host. Each gets its own key, which can do only what that binding needs:

```typescript
Effect.gen(function* () {
  const sandboxes = yield* Boat.CreateSandbox();
  return {
    fetch: Effect.gen(function* () {
      const running = yield* sandboxes.list({ labels: { pool: "agents" } });
      return yield* HttpServerResponse.json({ count: running.length });
    }),
  };
}).pipe(Effect.provide(Boat.CreateSandboxHttp));
```

To start sandboxes with a project already set up, see [Snapshots & forks](snapshots.md).
