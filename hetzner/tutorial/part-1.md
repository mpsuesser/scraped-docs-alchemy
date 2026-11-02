---
url: https://alchemy.run/hetzner/tutorial/part-1
title: "Part 1: Your First Server"
description: "Install Alchemy, create a Stack with a Hetzner Server, and deploy it."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

In this first part you’ll install Alchemy and Effect, create a Stack with a Hetzner Cloud Server, deploy it, and SSH in — all in under five minutes.

## Prerequisites

- [Bun](https://bun.sh/) or Node.js 22+
- A Hetzner project and an API token — see [Setup](../setup.md) if you haven’t created those yet

## Create a project

Start with an empty directory and initialize a `package.json`:

```sh
mkdir my-app && cd my-app && pnpm init
```

## Install dependencies

Install `alchemy@latest`, `@alchemy.run/hetzner`, and `effect`:

```sh
pnpm add alchemy@latest @alchemy.run/hetzner@latest effect @effect/platform-bun @effect/platform-node
```

## Create the Stack

Every Alchemy program starts with a `Stack` — a collection of Resources managed by Providers with state tracked between deploys.

Create an `alchemy.run.ts` file:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {Error ts(2345)  ― Argument of type '{ providers: Layer.Layer<never, never, never>; }' is not assignable to parameter of type 'StackProps<never>'.
  Property 'state' is missing in type '{ providers: Layer.Layer<never, never, never>; }' but required in type 'StackProps<never>'.
    providers: Layer.empty,
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

TypeScript is unhappy: the `state` property is required. Every Stack needs a **state store** so Alchemy can persist resource state between deploys and compute diffs against your infrastructure.

## Configure state

Hetzner has no state backend of its own, so we’ll keep state on disk with `Alchemy.localState()` — it writes to `.alchemy/` next to your code, no setup required:

```typescript
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    // we'll add resources here next
  }),
);
```

## Add a Server

Resources represent cloud infrastructure. Each resource is `yield*` -ed inside the Stack’s Effect generator.

Let’s declare a Server — Hetzner’s virtual machine — and observe the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Hetzner from "@alchemy.run/hetzner";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.empty,Error ts(2322)  ― Type 'Layer<never, never, never>' is not assignable to type 'Layer<NoInfer<Providers>, never, StackServices>'.
  Type 'Providers' is not assignable to type 'never'.
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const server = yield* Hetzner.Server("box", {
      serverType: "cx22",
      image: "ubuntu-24.04",
      location: "nbg1",
    });
  }),
);
```

The props map directly onto Hetzner’s console: `serverType` is the size (`cx22` = 2 vCPU / 4 GB), `image` is the OS, and `location` is the datacenter (`nbg1` is Nuremberg; `fsn1`, `hel1`, `ash`, `hil`, and `sin` also exist).

TypeScript is telling us that `Layer.empty` doesn’t provide `Hetzner.Providers` — the layer required by `Server`.

## Fix the Providers

Replace `Layer.empty` with `Hetzner.providers()` to resolve the type error:

```typescript
import * as Alchemy from "alchemy";
import * as Hetzner from "@alchemy.run/hetzner";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Hetzner.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const server = yield* Hetzner.Server("box", {
      serverType: "cx22",
      image: "ubuntu-24.04",
      location: "nbg1",
    });
  }),
);
```

Now the program type-checks. The providers layer tells Alchemy how to talk to the Hetzner Cloud API, and the type system ensures you never forget to wire it up.

## Add your SSH key

The server will boot without one, but registering your public key lets you SSH straight in (and stops Hetzner emailing you a root password). Declare an `SshKey` and pass it to the server:

```typescript
Effect.gen(function* () {
  const key = yield* Hetzner.SshKey("laptop", {
    publicKey: "ssh-ed25519 AAAA... you@laptop",
  });

  const server = yield* Hetzner.Server("box", {
    serverType: "cx22",
    image: "ubuntu-24.04",
    location: "nbg1",
    sshKeys: [key],
  });
}),
```

Paste your own key from `~/.ssh/id_ed25519.pub` (create one with `ssh-keygen -t ed25519` if you don’t have it). Passing the resolved `key` to `sshKeys` is your first **resource reference** — Alchemy sees the dependency and orders the deploy so the key exists before the server boots.

## Return Stack outputs

Stack outputs let you see important values after a deploy. Return an object from the generator to expose the server’s public addresses:

```typescript
const server = yield* Hetzner.Server("box", {
    serverType: "cx22",
    image: "ubuntu-24.04",
    location: "nbg1",
    sshKeys: [key],
  });

  return {
    ipv4: server.ipv4,
    ipv6: server.ipv6,
  };
}),
```

## Deploy

Run `alchemy deploy` to create the key and the server:

```sh
pnpm alchemy deploy
```

The first time you deploy, Alchemy prompts for Hetzner credentials — paste the API token you generated in [Setup](../setup.md). The token is verified and saved to your **`default`** [profile](../../environments/profiles.md), so you won’t be asked again.

```
Plan: 2 to create

+ laptop (Hetzner.SshKey)
+ box (Hetzner.Server)

Proceed?
◉ Yes ○ No
✓ laptop (Hetzner.SshKey) created
✓ box (Hetzner.Server) created
{
  ipv4: "203.0.113.10",
  ipv6: "2001:db8::1",
}
```

Alchemy shows a plan, asks for confirmation, creates both resources, and prints the stack outputs. A Hetzner server boots in about ten seconds.

## Verify it worked

The server is listed in your project in the [Hetzner Cloud Console](https://console.hetzner.com/), and you can SSH straight in:

```sh
ssh root@203.0.113.10
```

Run `alchemy deploy` again. Because nothing changed, both resources show as no-ops:

```
Plan: no changes

{
  ipv4: "203.0.113.10",
  ipv6: "2001:db8::1",
}
```

This is the core loop — declare resources in code, deploy, and Alchemy figures out what changed.

## Recap

You now have:

- An `alchemy.run.ts` with a Stack, an SSH key, and a Server
- A live VM running Ubuntu in Nuremberg, booted in seconds
- Stack outputs showing its public IPv4 and IPv6 addresses

In [Part 2](part-2.md), you’ll deploy code to this server — a `Hetzner.Service` that serves HTTP without you ever writing a systemd unit or a Dockerfile.
