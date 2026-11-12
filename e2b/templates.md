---
url: https://alchemy.run/e2b/templates
title: "Templates"
description: "Build the images your sandboxes boot from, from a registry image, a Dockerfile, or another template."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **template** is the image a sandbox boots from. E2B builds it on its side, so deploys need no Docker. Then it boots the image once, runs its start command, waits for its ready command to pass, and snapshots the running microVM. Every sandbox from the template starts from that snapshot, already past the build and the start, so it’s up in well under a second.

## A template is built once, then snapshotted

<svg viewBox="0 0 670 168" style="max-width: 770px" role="img" aria-label="A deploy builds the template once and snapshots it when readyCmd passes. Sandboxes created later start from the snapshot, with the start command already running."><line stroke-opacity="0.2" stroke="currentColor" x1="378" y1="24" x2="378" y2="162"></line><g><text fill="currentColor" x="4" y="50">MyTemplate</text> <text fill="currentColor" x="4" y="64">template</text> <g><rect stroke="currentColor" fill="none" x="98" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="182" y="56" text-anchor="middle">build image</text></g> <g><rect stroke="currentColor" fill="none" x="266" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="322" y="56" text-anchor="middle">startCmd</text></g> <g><rect stroke="currentColor" fill="none" x="378" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="434" y="56" text-anchor="middle">snapshot</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="670" y2="74"></line><text fill="currentColor" x="4" y="94">user-42</text> <text fill="currentColor" x="4" y="108">sandbox</text> <g><rect stroke="currentColor" fill="none" x="518" y="84" width="140" height="24" rx="5"></rect><text fill="currentColor" x="588" y="100" text-anchor="middle">running</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="670" y2="118"></line><text fill="currentColor" x="4" y="138">user-43</text> <text fill="currentColor" x="4" y="152">sandbox</text> <g><rect stroke="currentColor" fill="none" x="574" y="128" width="84" height="24" rx="5"></rect><text fill="currentColor" x="616" y="144" text-anchor="middle">running</text></g></g> <text fill="currentColor" x="378" y="18" text-anchor="middle">readyCmd passes</text></svg> <svg viewBox="0 0 330 213" style="max-width: 379px" role="img" aria-label="A deploy builds the template once and snapshots it when readyCmd passes. Sandboxes created later start from the snapshot, with the start command already running."><line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="24" x2="159" y2="207"></line><g><text fill="currentColor" x="0" y="44">MyTemplate <tspan fill="currentColor" dx="8">template</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="70" text-anchor="middle">build image</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="54" width="63.60000000000001" height="24" rx="5"></rect><text fill="currentColor" x="127.19999999999999" y="70" text-anchor="middle">startCmd</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="190.8" y="70" text-anchor="middle">snapshot</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">user-42 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="238.5" y="108" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="278.25" y="124" text-anchor="middle">running</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">user-43 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="270.3" y="162" width="47.69999999999999" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="198" text-anchor="start">running</text></g></g> <text fill="currentColor" x="159" y="18" text-anchor="middle">readyCmd passes</text></svg>

A deploy builds the template once and snapshots it when readyCmd passes. Sandboxes created later start from the snapshot, with the start command already running.

## Build from a registry image

```typescript
import * as E2B from "@alchemy.run/e2b";

export const Python = E2B.Template("Python", { image: "python:3.12-slim" });
```

## Build from a Dockerfile

```typescript
import * as Dockerfile from "@alchemy.run/docker/Dockerfile";
import * as E2B from "@alchemy.run/e2b";

export const MyTemplate = E2B.Template("MyTemplate", {
  dockerfile: Dockerfile.inline\`
    FROM node:22-bookworm
    RUN apt-get update && apt-get install -y git ripgrep
    WORKDIR /work
  \`,
});
```

Point `dockerfile` at a file instead to use its directory as the build context. `COPY` sources are uploaded once per content, so a deploy uploads only files that changed.

E2B’s builder runs one stage. A Dockerfile with several `FROM` s, `COPY --from`, `ADD <url>`, heredocs, `RUN --mount`, or `SHELL` fails at plan time with `UnsupportedDockerfile`, listing the lines, before anything is created. `EXPOSE`, `LABEL`, `VOLUME`, `HEALTHCHECK`, and `STOPSIGNAL` are ignored, and an `ARG` becomes an environment variable.

## Build on another template

```typescript
export const Tools = E2B.Template("Tools", {
  fromTemplate: "base",
  run: ["pip install pandas numpy"],
});
```

Building on a template reuses its cached layers.

## Size the microVM

```typescript
export const Big = E2B.Template("Big", { image: "node:22-bookworm", cpuCount: 8, memoryMB: 16384 });
```

Every sandbox from the template gets these resources.

## Start a process before the snapshot

```typescript
export const Preview = E2B.Template("Preview", {
  image: "node:22-bookworm",
  run: ["npm create vite@latest app -- --template react", "cd app && npm install"],
  startCmd: "cd /home/user/app && npm run dev -- --host 0.0.0.0 --port 3000",
  readyCmd: "curl -sf http://localhost:3000",
});
```

E2B snapshots the template once `readyCmd` succeeds, so every sandbox starts with the dev server up. A Dockerfile’s `CMD` does the same. To run your own Effect program this way, see [Template programs](programs.md).

## Pull a private base image

```typescript
export const Private = E2B.Template("Private", {
  image: "ghcr.io/my-org/base:1",
  registry: { username: "my-org", password: Config.Redacted("GHCR_TOKEN") },
});
```

E2B uses the credentials to pull the base image only. ECR (`awsAccessKeyId`, `awsSecretAccessKey`, `awsRegion`) and Artifact Registry (`serviceAccountJson`) work the same way.

## Build templates while your app runs

```typescript
const templates = yield* E2B.CreateTemplate();
// in fetch:
const template = yield* templates.getOrBuild({
  name: \`repo-${slug}\`,
  image: "node:22-bookworm",
  files: [{ path: "/home/user/app/package.json", contents: packageJson }],
  run: ["cd /home/user/app && npm install"],
});
```

A runtime build takes E2B’s own build inputs: a base `image` or `fromTemplate`, then `env`, `files` from memory, and `run` steps. `getOrBuild` builds only when no ready build of the same inputs exists. Delete runtime templates when you no longer need them: destroying the Worker does not.

## A changed template is rebuilt

A deploy starts a new build only when the inputs change; sandboxes already running keep the old build. Changing `name` replaces the template.
