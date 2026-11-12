---
url: https://alchemy.run/e2b/tutorial/part-4
title: "Part 4: Update the Template"
description: "Change the E2B template, redeploy, and see the snapshot rebuilt on it, while existing sandboxes keep what they started with."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Add a tool to the image

Install pnpm in the template:

```typescript
export const MyTemplate = E2B.Template("MyTemplate", {
  dockerfile: Dockerfile.inline\`
    FROM node:22-bookworm
    RUN apt-get update && apt-get install -y git
    RUN npm install -g pnpm
  \`,
  cpuCount: 2,
  memoryMB: 2048,
});
```

## Deploy the new base

```sh
pnpm alchemy deploy
```

The Dockerfile changed, so E2B builds `MyTemplate` again. `Warm` starts from `MyTemplate`, so Alchemy snapshots it again on the new build, and clones the repository into it. The Worker is unchanged and isn’t redeployed.

```sh
curl -X POST "$URL/run" --data 'pnpm --version'
```

## Running sandboxes keep what they started with

A sandbox keeps the snapshot it booted from. Users whose sandbox is paused keep it, with their changes and without pnpm. Sandboxes started after the deploy get the new base. To move a user onto it, kill their sandbox; their next request starts a new one from `Warm`.

## Next

[Part 5](part-5.md) tests it.
