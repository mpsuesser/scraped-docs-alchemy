---
url: https://alchemy.run/modal/sandboxes/images
title: "Images"
description: "Build Modal images from a registry image, a Dockerfile, or another image, and start sandboxes with a repository already installed."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

An **image** is the filesystem a Modal container starts from. Modal builds it on its own builders, so a deploy needs no Docker on your machine, and it caches each build by its inputs.

## An image builds on another

Each image starts from a registry image or from another image, and adds its own layer. A warm base runs your setup once, at deploy, so every sandbox starts with the work done:

<svg viewBox="0 0 682.4 128" style="max-width: 785px" role="img" aria-label="Node adds git to a registry image. Warm starts from Node and installs the repository. Sandboxes from Warm start with both layers in place."><g><text fill="currentColor" x="4" y="30">Node</text> <text fill="currentColor" x="4" y="44">node:22-bookworm</text> <line stroke-opacity="0.2" stroke="currentColor" x1="126.4" y1="30" x2="666.4" y2="30"></line></g><g><text fill="currentColor" x="4" y="94">Warm</text> <text fill="currentColor" x="4" y="108">from Node</text> <path stroke="currentColor" fill="none" d="M 306.4 30 C 324.4 30, 314.4 94, 332.4 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="332.4" y1="94" x2="666.4" y2="94" style="stroke-width: 3"></line></g><g><circle stroke="currentColor" fill="none" cx="216.4" cy="30" r="5.5"></circle><text fill="currentColor" x="216.4" y="19" text-anchor="middle">apt-get install git</text></g> <g><circle stroke="currentColor" fill="none" cx="441.4" cy="94" r="5.5"></circle><text fill="currentColor" x="441.4" y="83" text-anchor="middle">git clone · pnpm install</text></g></svg> <svg viewBox="0 0 328 158" style="max-width: 377px" role="img" aria-label="Node adds git to a registry image. Warm starts from Node and installs the repository. Sandboxes from Warm start with both layers in place."><g><text fill="currentColor" x="0" y="16">Node <tspan fill="currentColor" dx="8">node:22-bookworm</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46"></line></g><g><text fill="currentColor" x="135.33333333333331" y="94">Warm <tspan fill="currentColor" dx="8">from Node</tspan></text> <path stroke="currentColor" fill="none" d="M 109.33333333333333 46 C 127.33333333333333 46, 117.33333333333333 124, 135.33333333333331 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="135.33333333333331" y1="124" x2="312" y2="124" style="stroke-width: 3"></line></g><g><circle stroke="currentColor" fill="none" cx="58.666666666666664" cy="46" r="5.5"></circle><text fill="currentColor" x="2" y="35" text-anchor="start">apt-get install git</text></g> <g><circle stroke="currentColor" fill="none" cx="185.33333333333334" cy="124" r="5.5"></circle><text fill="currentColor" x="108.53333333333333" y="113" text-anchor="start">git clone · pnpm install</text></g></svg>

Node adds git to a registry image. Warm starts from Node and installs the repository. Sandboxes from Warm start with both layers in place.

## Build from a registry image

```typescript
import * as Modal from "@alchemy.run/modal";

export const Coder = Modal.App("Coder", {});

export const Node = Modal.Image("Node", {
  app: Coder,
  image: "node:22-bookworm",
  commands: ["RUN apt-get update && apt-get install -y git ripgrep"],
});
```

`commands` are Dockerfile lines, run on top of the registry image.

## Build from a Dockerfile

```typescript
import * as Dockerfile from "@alchemy.run/docker/Dockerfile";
import * as Modal from "@alchemy.run/modal";

export const Coder = Modal.App("Coder", {});

export const Python = Modal.Image("Python", {
  app: Coder,
  dockerfile: Dockerfile.inline\`
    FROM python:3.12-slim
    RUN pip install pandas
  \`,
});
```

Modal’s builder isn’t Docker. Multi-stage builds, BuildKit mounts, `EXPOSE`, `HEALTHCHECK`, `LABEL`, `ONBUILD`, `STOPSIGNAL` and `VOLUME` fail at plan, naming the line. `USER` is ignored: containers run as root.

A Dockerfile path works too: `dockerfile: "./sandbox/Dockerfile"`. Files its `COPY` lines read are uploaded from the Dockerfile’s directory, or from `context`.

## Install a repository into a warm base

```typescript
export const Warm = Modal.Image("Warm", {
  app: Coder,
  from: Node,
  run: [
    "git clone https://github.com/alchemy-run/alchemy /work",
    "cd /work && corepack enable && pnpm install",
  ],
});
```

`run` executes in a sandbox at deploy, then saves its files as the image. Changing `run` or the base builds a new one.

## Build images while your app runs

For an image per repository or per user, bind `CreateImage` and build at run time:

```typescript
const images = yield* Modal.CreateImage(Coder);
const image = yield* images.build({
  image: "node:22-bookworm",
  commands: [\`RUN git clone https://github.com/${repo} /work\`],
});
const sandbox = yield* sandboxes.create({ image });
```

Modal caches builds by content, so the same inputs return the same image.
