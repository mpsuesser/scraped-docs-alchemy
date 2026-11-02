---
url: https://alchemy.run/fly/compute/apps
title: "Apps"
description: "A global namespace containing Machines, Services, Secrets, IPs, and certificates."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

A [`Fly.App`](https://alchemy.run/providers/fly/reference/app#app) is a global namespace in your
account. It contains Machines, Services, Secrets, IPs, and
certificates.

A [Service](services.md) creates and manages its own App,
so you do not declare one to deploy a Service. Declare a `Fly.App`
when you need one directly:

- [Machines](machines.md) run in an App you declare.
- [`Fly.Secret`](../data/secrets.md), `SecretKey`,
  [`Fly.Certificate`](../networking.md#use-your-own-hostname), and
  [`IpAssignment`](../networking.md#addresses-for-a-shared-app) attach
  to a `Fly.App`, not to a Service's own App.
- Several Services share one hostname, secrets, or domain — see
  [Group Services in one App](services.md#group-services-in-one-app).

## Create an App

Alchemy generates a unique name unless you pass one.

```typescript
const site = yield* Fly.App("Site");
```

`url` is `https://{appName}.fly.dev`. Nothing answers there until a
[Machine](machines.md) or a grouped
[Service](services.md#group-services-in-one-app) publishes
a proxy service and the App has an
[IP assignment](../networking.md#addresses-for-a-shared-app).

:::note
Prefer omitting `name` in tests and CI so names stay unique and
reclaimable.
:::

## A stable hostname

Pass `name` when you need a stable `fly.dev` hostname.

```typescript
const site = yield* Fly.App("Site", {
  name: "my-site",
});
```

:::caution[Changing `name` replaces the App]
Fly cannot have two Apps with the same name. Alchemy deletes the
old App first, then creates the new one.
:::

## Organization

Org defaults to the current token. Pass `orgSlug` to pin it.

```diff lang="typescript"
const site = yield* Fly.App("Site", {
  name: "my-site",
+  orgSlug: "my-org",
});
```

:::caution[Changing `orgSlug` replaces the App]
The App is created in the new org. The old App is deleted.
:::

## Subdomains

`enableSubdomains: true` turns on `*.{appName}.fly.dev`.

```diff lang="typescript"
const site = yield* Fly.App("Site", {
  name: "my-site",
  orgSlug: "my-org",
+  enableSubdomains: true,
});
```

:::note[Create-only]
Flipping `enableSubdomains` later is ignored.
:::

## Isolated network

`network` is an optional 6PN name.

```diff lang="typescript"
const site = yield* Fly.App("Site", {
  name: "my-site",
  orgSlug: "my-org",
  enableSubdomains: true,
+  network: "private",
});
```

:::caution[Changing `network` replaces the App]
The App is recreated on the new network.
:::

## Module-scope declarations

Declare the App once. Pass it into every child.

```typescript
// src/app.ts
import * as Fly from "@alchemy.run/fly";

export const Site = Fly.App("Site");
```

Resource-valued props accept the resource or an Effect producing
it. Do not unwrap it just to pass it along.

## Destroy

`alchemy destroy` deletes children, then the App.

## Where next

[Services](services.md) own their App by default and join
one with `app`. [Machines](machines.md) run a raw image in
an App. [IPs & certificates](../networking.md) covers addresses and
domains on an App. The [`App` reference](https://alchemy.run/providers/fly/reference/app#app) lists
every prop.
