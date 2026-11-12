---
url: https://alchemy.run/modal/tutorial/part-4
title: "Part 4: Update the Image"
description: "Change the warm Modal image, redeploy, and see new sandboxes start from the new image while users keep their files."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Add a command

Install pnpm into the warm image:

```typescript
export const Warm = Modal.Image("Warm", {
  app: Coder,
  from: Node,
  run: ["git clone --depth 1 https://github.com/sindresorhus/is /work"],
  run: ["git clone --depth 1 https://github.com/sindresorhus/is /work", "npm install -g pnpm"],
});
```

## Deploy the new base

```sh
pnpm alchemy deploy
```

`run` changed, so Modal builds a new image: it runs the commands again in a fresh sandbox and saves the result. The Worker is unchanged and isn’t redeployed.

```sh
curl -X POST "$URL/users/ada/run" -d 'pnpm --version'
```

## Users keep their files

A sandbox that’s already running keeps the image it booted from, until its command exits. Every sandbox started after the deploy boots from the new `Warm`, and mounts the same user volume, so Ada’s notes are still there, now next to pnpm.

## Next

[Part 5](part-5.md) tests it.
