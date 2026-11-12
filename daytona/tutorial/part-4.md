---
url: https://alchemy.run/daytona/tutorial/part-4
title: "Part 4: Update the Snapshot"
description: "Move the Daytona snapshots to a new Node.js version, redeploy, and see new sandboxes start from it while existing ones keep theirs."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Change the image

Move the snapshots to Node.js 24:

```typescript
export const Base = Daytona.Snapshot("Base", {
  image: "node:22-bookworm",
  image: "node:24-bookworm",
  cpu: 1,
  memory: 1,
  disk: 3,
});
```

## Deploy the new base

```sh
pnpm alchemy deploy
```

A snapshot can’t be changed, so a new image replaces `Base`. `Warm` starts from `Base`, so it’s captured again on the new image, with the repository cloned into it. The Worker is unchanged and isn’t redeployed.

```sh
curl -X POST "$URL/run" -d 'node --version'
```

## Running sandboxes keep what they started with

A sandbox keeps the disk it booted with. Users who already have a sandbox keep it, with their changes and Node.js 22. Sandboxes started after the deploy get the new base. To move a user onto it, delete their sandbox; their next request starts a new one from `Warm`.

## Next

[Part 5](part-5.md) tests it.
