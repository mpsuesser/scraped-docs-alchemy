---
url: https://alchemy.run/daytona/sandboxes/snapshots
title: "Snapshots"
description: "Pull an image, build a Dockerfile on Daytona, or capture a warm base that sandboxes start from."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **snapshot** is the image a sandbox boots from, with the size every sandbox from it gets: CPU, memory, disk and GPUs. Daytona builds it once and every sandbox starts from a copy. Snapshots can’t be changed: changing what one is built from replaces it.

## Every sandbox starts from a copy

Daytona makes a snapshot from an image, from a Dockerfile, or from another snapshot with commands run in it. That last one is how you get a **warm base**: a snapshot with a repository already cloned and its dependencies installed, so sandboxes start ready to work.

<svg viewBox="0 0 721.8 198" style="max-width: 830px" role="img" aria-label="Deploy starts a sandbox from Base, runs the commands, captures its disk as Warm and deletes the sandbox. Every sandbox created from Warm starts with the repository cloned and installed."><line stroke-opacity="0.2" stroke="currentColor" x1="149.8" y1="24" x2="149.8" y2="192"></line><g><text fill="currentColor" x="4" y="56">sandbox from Base</text> <g><rect stroke="currentColor" fill="none" x="149.8" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="205.8" y="56" text-anchor="middle">git clone</text></g> <g><rect stroke="currentColor" fill="none" x="267.40000000000003" y="40" width="139.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="337.4" y="56" text-anchor="middle">pnpm install</text></g> <g><rect stroke="currentColor" fill="none" x="413" y="40" width="89.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="457.8" y="56" text-anchor="middle">capture</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="721.8" y2="74"></line><text fill="currentColor" x="4" y="100">Warm</text> <g><rect stroke="currentColor" fill="none" x="508.20000000000005" y="84" width="201.5999999999999" height="24" rx="5"></rect><text fill="currentColor" x="609" y="100" text-anchor="middle">snapshot Warm</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="721.8" y2="118"></line><rect stroke="currentColor" fill="none" x="141.8" y="121" width="576" height="68" rx="8"></rect><text fill="currentColor" x="4" y="159">sandboxes</text> <g><rect stroke="currentColor" fill="none" x="530.5999999999999" y="128" width="134.4000000000001" height="24" rx="5"></rect><text fill="currentColor" x="597.8" y="144" text-anchor="middle">ready at boot</text></g> <g><rect stroke="currentColor" fill="none" x="575.4000000000001" y="158" width="134.39999999999986" height="24" rx="5"></rect><text fill="currentColor" x="642.6" y="174" text-anchor="middle">ready at boot</text></g></g> <text fill="currentColor" x="149.8" y="18" text-anchor="middle">deploy</text></svg> <svg viewBox="0 0 330 273" style="max-width: 379px" role="img" aria-label="Deploy starts a sandbox from Base, runs the commands, captures its disk as Warm and deletes the sandbox. Every sandbox created from Warm starts with the repository cloned and installed."><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="24" x2="0" y2="267"></line><g><text fill="currentColor" x="0" y="44">sandbox from Base</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="63.6" height="24" rx="5"></rect><text fill="currentColor" x="0" y="90" text-anchor="start">git clone</text></g> <g><rect stroke="currentColor" fill="none" x="66.78" y="54" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="66.78" y="90" text-anchor="start">pnpm install</text></g> <g><rect stroke="currentColor" fill="none" x="149.46" y="54" width="50.879999999999995" height="24" rx="5"></rect><text fill="currentColor" x="206.34" y="70" text-anchor="start">capture</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">Warm</text> <g><rect stroke="currentColor" fill="none" x="203.52" y="123" width="114.47999999999999" height="24" rx="5"></rect><text fill="currentColor" x="260.76" y="139" text-anchor="middle">snapshot Warm</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="153" x2="330" y2="153"></line><rect stroke="currentColor" fill="none" x="0" y="174" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="167">sandboxes</text> <g><rect stroke="currentColor" fill="none" x="216.23999999999998" y="177" width="76.32000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.23999999999998" y="213" text-anchor="start">ready at boot</text></g> <g><rect stroke="currentColor" fill="none" x="241.68" y="222" width="76.32" height="24" rx="5"></rect><text fill="currentColor" x="234.8" y="258" text-anchor="start">ready at boot</text></g></g> <text fill="currentColor" x="0" y="18" text-anchor="start">deploy</text></svg>

Deploy starts a sandbox from Base, runs the commands, captures its disk as Warm and deletes the sandbox. Every sandbox created from Warm starts with the repository cloned and installed.

## Pull an image

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Node = Daytona.Snapshot("Node", {
  image: "node:22-bookworm",
  cpu: 2,
  memory: 4,
  disk: 10,
});
```

Use a fixed tag or digest. Daytona rejects `latest`, `lts` and `stable`, and deploy fails before calling it.

## Pull from a private registry

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Config from "effect/Config";

export const Ghcr = Daytona.Registry("Ghcr", {
  url: "ghcr.io",
  username: "alchemy-run",
  password: Config.Redacted("GHCR_TOKEN"),
});

export const App = Daytona.Snapshot("App", {
  image: "ghcr.io/alchemy-run/app:1.4.0",
  registry: Ghcr,
});
```

Daytona picks the registry whose `url` matches the image’s host.

## Build a Dockerfile on Daytona

```typescript
import * as Dockerfile from "@alchemy.run/docker/Dockerfile";
import * as Daytona from "@alchemy.run/daytona";

export const Tools = Daytona.Snapshot("Tools", {
  dockerfile: Dockerfile.inline\`
    FROM node:22-bookworm
    RUN apt-get update && apt-get install -y git ripgrep
  \`,
});
```

Daytona’s builders run the build; you don’t need Docker.

## Build a directory with its Dockerfile

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const App = Daytona.Snapshot("App", {
  context: "./sandbox",
  build: { args: { NODE_ENV: "production" } },
});
```

The files `./sandbox/Dockerfile` copies are uploaded to Daytona, minus anything in `.dockerignore`. Changing one of them rebuilds the snapshot. Build arguments are written into the Dockerfile’s `ARG` defaults. Daytona builds for `linux/amd64` only. Daytona’s builders can’t receive build secrets, so setting `build.secrets` builds with your local Docker and pushes the image to Daytona instead.

## Capture a warm base

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Base = Daytona.Snapshot("Base", { image: "node:22-bookworm" });

export const Warm = Daytona.Snapshot("Warm", {
  from: Base,
  run: [
    "git clone https://github.com/alchemy-run/alchemy /home/daytona/alchemy",
    "cd /home/daytona/alchemy && corepack enable && pnpm install",
  ],
});
```

Deploy starts a sandbox from `Base`, runs the commands, captures its disk as `Warm`, and deletes the sandbox. Sandboxes created from `Warm` start with the repository cloned and installed.

## Keep sandboxes warm

```typescript
export const Pool = Daytona.WarmPool("Pool", { snapshot: Warm, size: 4 });
```

Daytona keeps four started sandboxes of `Warm` ready and hands one out on each create. They are billed while they wait.

## Build snapshots at runtime

```typescript
// construction
const images = yield* Daytona.CreateSnapshot({ prefix: "repo-" });
const warm = yield* Warm;
// per request
const name = \`repo-${owner}-${repo}\`;
const base =
  (yield* images.get(name)) ??
  (yield* images.create({
    name,
    from: warm.name,
    run: [\`git clone https://github.com/${owner}/${repo} /home/daytona/repo\`],
  }));
```

`CreateSnapshot` builds a base per repository the first time a user needs one. Snapshots have only a name, so `prefix` is how you find the ones it built; they are not deleted when the stack is destroyed.

## Snapshot a running sandbox

```typescript
yield* sandbox.createSandboxSnapshot({ name: \`checkpoint-${task}\` });
```

## Unused snapshots are deactivated

Daytona deactivates a snapshot nobody used for two weeks. Deploy reactivates it; at runtime, activate it before you create from it:

```typescript
yield* images.activate(name);
```
