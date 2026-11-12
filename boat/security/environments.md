---
url: https://alchemy.run/boat/security/environments
title: "Environments & secrets"
description: "Boat environments — the variables, secret files and credentials a sandbox starts with, versioned on every change."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

An **environment** is what a sandbox starts with: environment variables, secret files, and which of your account’s credentials it gets — a GitHub token for your repositories, a Boat key of its own, the AI keys of a [credential profile](credential-profiles.md). Your account has a default environment, `base`, which sandboxes get unless they name another.

## Every change makes a new version

Every change to an environment makes a new **version**. A sandbox keeps the version it started on, so a change reaches new sandboxes, not running ones; the dashboard upgrades running ones.

<svg viewBox="0 0 666.4 212" style="max-width: 766px" role="img" aria-label="Rotating TOKEN made version 2. task-3 started after it and has the new token; task-1 and task-2 keep version 1 for as long as they run."><line stroke-opacity="0.2" stroke="currentColor" x1="374.4" y1="24" x2="374.4" y2="206"></line><g><text fill="currentColor" x="4" y="50">MyEnv</text> <text fill="currentColor" x="4" y="64">environment</text> <g><rect stroke="currentColor" fill="none" x="94.4" y="40" width="280" height="24" rx="5"></rect><text fill="currentColor" x="234.4" y="56" text-anchor="middle">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="374.4" y="40" width="280" height="24" rx="5"></rect><text fill="currentColor" x="514.4" y="56" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="666.4" y2="74"></line><text fill="currentColor" x="4" y="94">task-1</text> <text fill="currentColor" x="4" y="108">sandbox</text> <g><rect stroke="currentColor" fill="none" x="150.4" y="84" width="504" height="24" rx="5"></rect><text fill="currentColor" x="402.4" y="100" text-anchor="middle">on version 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="666.4" y2="118"></line><text fill="currentColor" x="4" y="138">task-2</text> <text fill="currentColor" x="4" y="152">sandbox</text> <g><rect stroke="currentColor" fill="none" x="262.4" y="128" width="336" height="24" rx="5"></rect><text fill="currentColor" x="430.4" y="144" text-anchor="middle">on version 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="162" x2="666.4" y2="162"></line><text fill="currentColor" x="4" y="182">task-3</text> <text fill="currentColor" x="4" y="196">sandbox</text> <g><rect stroke="currentColor" fill="none" x="430.4" y="172" width="224" height="24" rx="5"></rect><text fill="currentColor" x="542.4" y="188" text-anchor="middle">on version 2</text></g></g> <text fill="currentColor" x="374.4" y="18" text-anchor="middle">TOKEN rotated</text></svg> <svg viewBox="0 0 330 252" style="max-width: 379px" role="img" aria-label="Rotating TOKEN made version 2. task-3 started after it and has the new token; task-1 and task-2 keep version 1 for as long as they run."><line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="24" x2="159" y2="246"></line><g><text fill="currentColor" x="0" y="44">MyEnv <tspan fill="currentColor" dx="8">environment</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="159" height="24" rx="5"></rect><text fill="currentColor" x="79.5" y="70" text-anchor="middle">version 1</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="159" height="24" rx="5"></rect><text fill="currentColor" x="238.5" y="70" text-anchor="middle">version 2</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">task-1 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="108" width="286.2" height="24" rx="5"></rect><text fill="currentColor" x="174.9" y="124" text-anchor="middle">on version 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">task-2 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="162" width="190.8" height="24" rx="5"></rect><text fill="currentColor" x="190.8" y="178" text-anchor="middle">on version 1</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="192" x2="330" y2="192"></line><text fill="currentColor" x="0" y="206">task-3 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="216" width="127.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="232" text-anchor="middle">on version 2</text></g></g> <text fill="currentColor" x="159" y="18" text-anchor="middle">TOKEN rotated</text></svg>

Rotating TOKEN made version 2. task-3 started after it and has the new token; task-1 and task-2 keep version 1 for as long as they run.

## Declare an environment with variables and secret files

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Config from "effect/Config";

export const MyEnv = Boat.Environment("MyEnv", {
  env: { NODE_ENV: "production", TOKEN: Config.Redacted("AGENT_TOKEN") },
  secretFiles: [{ path: ".npmrc", contents: Config.Redacted("NPMRC") }],
});
```

Secret file paths are under the home directory: `.npmrc`, `~/.npmrc` and `/home/user/.npmrc` are the same file. Values stay `Redacted` in state. Boat stores variables as a `.env` file, which can’t hold a value with a newline, or with both `'` and `"`; such a value fails with `InvalidEnvironmentValue`.

An unchanged deploy makes no new version. A changed one updates only what differs.

## Start sandboxes on an environment

Pass the environment to a declared sandbox:

```typescript
export const Box = Boat.Sandbox("Box", { environment: MyEnv });
```

A sandbox your code creates names it:

```typescript
const env = yield* MyEnv;
const sandbox = yield* sandboxes.create({ environment: env.name, autoStop: "10 minutes" });
```

## Choose which credentials it passes

An environment you declare passes its own `env` and `secretFiles`, and nothing of your account. Each of your credentials is a flag you turn on:

```typescript
export const Mine = Boat.Environment("Mine", {
  passGithub: true, // your repositories, and a GitHub token
  passSandboxCredentials: true, // a Boat key, to run \`boat\` inside the sandbox
});
```

Boat’s own default is to pass everything, which is what `base` does. Alchemy sets each flag explicitly, so a sandbox holds only what its environment asks for.

AI keys come from a [credential profile](credential-profiles.md). Name one, and its keys reach the environment’s sandboxes:

```typescript
export const WithAi = Boat.Environment("WithAi", { credentialProfile: MyProfile });
```

## Give sandboxes to your users

`safeForThirdParties` passes nothing of your account, whatever the flags say. Use it for sandboxes someone else gets a shell in:

```typescript
export const Tenants = Boat.Environment("Tenants", { safeForThirdParties: true });
```

Such sandboxes can still call AI providers through Boat’s proxy, holding placeholders instead of keys. See [Keep real keys out of the sandbox](credential-profiles.md#keep-real-keys-out-of-the-sandbox).

A single sandbox with none of the account’s secrets doesn’t need an environment: pass `noEnv: true` on it.

## Use an environment the Stack doesn’t manage

`Boat.Environment.lookup` finds an environment by name, like the `base` one every account has. The Stack reads it and never changes or deletes it:

```typescript
export const Box = Boat.Sandbox("Box", {
  environment: Boat.Environment.lookup({ name: "base" }),
});
```

## Pull values from a secrets manager

`.env` is only the default. A Stack’s `secrets` option reads the same `Config` values from a secrets manager instead, like Doppler or Infisical, so production keys never sit in a file on anyone’s machine. Pick the source per stage:

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Doppler from "@alchemy.run/doppler";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { MyEnv } from "./src/MyEnv.ts";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Boat.providers()),
    state: Cloudflare.state(),
    secrets: ({ stage }) =>
      stage === "prod"
        ? Doppler.Secrets({ project: "my-app", config: "prd" })
        : Alchemy.Secrets.DotEnv(),
  },
  Effect.gen(function* () {
    const env = yield* MyEnv;
    return { environment: env.name };
  }),
);
```

`Config.Redacted("AGENT_TOKEN")` doesn’t change. See [Secret providers](../../environments/secret-providers.md).

To call a server running in a sandbox, see [Ports & URLs](../sandboxes/ports.md).
