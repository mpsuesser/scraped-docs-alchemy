---
url: https://alchemy.run/archil/setup
title: "Setup"
description: "Install the Archil provider, create an API key, and store it in a profile."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything you need before deploying to Archil: the packages, an Archil account, and an API key stored in a profile.

## Prerequisites

- [Node.js](https://nodejs.org/) 24.11.1+ or [Bun](https://bun.sh/)
- An [Archil](https://console.archil.com/) account
- [Docker](https://docs.docker.com/get-docker/), only to build an [Image](compute/images.md) from a Dockerfile. Programs and public images don’t need it.

## Install

```sh
pnpm add alchemy@latest @alchemy.run/archil@latest effect @effect/platform-bun @effect/platform-node
```

## Add the provider

Add `Archil.providers()` next to your other providers:

```typescript
import * as Alchemy from "alchemy";
import * as Archil from "@alchemy.run/archil";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  { providers: Archil.providers(), state: Alchemy.localState() },
  Effect.gen(function* () {
    // your resources
  }),
);
```

With a second cloud, merge them: `Layer.mergeAll(Cloudflare.providers(), Archil.providers())`.

## Create an API key

Sign in at [console.archil.com](https://console.archil.com/), select the account or organization you deploy to, and create a key under **API Keys**. It starts with `key-`. Archil shows it once.

## Store it in a profile

```sh
alchemy profile edit --add Archil
```

Paste the key, and pick the region your disks live in (`aws-us-east-1` by default). Enter your organization id only if you keep disks on your own S3 bucket.

## Deploy from CI

Set the key in the environment. It takes precedence over any profile:

```sh
ARCHIL_API_KEY=key-... ARCHIL_REGION=aws-us-east-1 alchemy deploy
```

See [Organization and keys](organization-and-keys.md) for profiles per environment and what each host gets instead of your key.
