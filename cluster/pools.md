---
url: https://alchemy.run/cluster/pools
title: "Pools"
description: "How a pool of Effect Cluster runners hosts your entities, workflows, singletons and crons — what's bundled together, shard groups, the gateway, and how to split work across pools."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **pool** is a group of **runners**, the processes that run your code. A pool is a Layer: provide it to an implementation, and that implementation runs on every runner in the pool. Each host names its pool after what it runs on, like `ClusterDocker.SwarmPool` or `ClusterAWS.EcsPool`.

```typescript
import * as ClusterDocker from "@alchemy.run/effect-cluster/Docker";
import * as Layer from "effect/Layer";

export const Database = ClusterDocker.Database("Cluster");

export const Default = ClusterDocker.SwarmPool("Default", { replicas: 2 }).pipe(
  Layer.provide(Database),
);
```

## One pool, one program

Every class provided the same pool is bundled into one Node program, built into one image, and run on every runner in the pool. A change to any of them builds a new image, and the host replaces the runners, a few at a time.

<svg viewBox="0 0 628 296" style="max-width: 722px" role="img" aria-label="Every runner in a pool runs the same classes."><defs><marker id="alc-dag-arrow-mlevme" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 C 201 56, 201 148, 238 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-mlevme)"></path></g><g><path d="M 164 148 C 201 148, 201 148, 238 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-mlevme)"></path></g><g><path d="M 164 240 C 201 240, 201 148, 238 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-mlevme)"></path></g><g><path d="M 384 148 C 421 148, 421 106, 458 106" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-mlevme)"></path></g><g><path d="M 384 148 C 421 148, 421 190, 458 190" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-mlevme)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Room</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">entity</text></g> <g transform="translate(24, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Invite</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">workflow</text></g> <g transform="translate(24, 208)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Cleanup</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">cron</text></g> <g transform="translate(244, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Default</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">one image</text></g> <g transform="translate(464, 78)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Runner 1</text></g> <g transform="translate(464, 162)"><rect stroke="currentColor" fill="none" width="140" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="29" text-anchor="middle" dominant-baseline="middle">Runner 2</text></g></svg> <svg viewBox="0 0 364 332" style="max-width: 419px" role="img" aria-label="Every runner in a pool runs the same classes."><defs><marker id="alc-dag-arrow-ywrpwk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 58 74 C 58 101, 182 101, 182 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ywrpwk)"></path></g><g><path d="M 182 74 C 182 101, 182 101, 182 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ywrpwk)"></path></g><g><path d="M 306 74 C 306 101, 182 101, 182 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ywrpwk)"></path></g><g><path d="M 182 198 C 182 225, 120 225, 120 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ywrpwk)"></path></g><g><path d="M 182 198 C 182 225, 244 225, 244 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-ywrpwk)"></path></g><g transform="translate(10, 10)"><rect stroke="currentColor" fill="none" width="96" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="28" text-anchor="middle" dominant-baseline="middle">Room</text> <text fill="currentColor" x="48" y="46" text-anchor="middle" dominant-baseline="middle">entity</text></g> <g transform="translate(134, 10)"><rect stroke="currentColor" fill="none" width="96" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="28" text-anchor="middle" dominant-baseline="middle">Invite</text> <text fill="currentColor" x="48" y="46" text-anchor="middle" dominant-baseline="middle">workflow</text></g> <g transform="translate(258, 10)"><rect stroke="currentColor" fill="none" width="96" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="28" text-anchor="middle" dominant-baseline="middle">Cleanup</text> <text fill="currentColor" x="48" y="46" text-anchor="middle" dominant-baseline="middle">cron</text></g> <g transform="translate(134, 134)"><rect stroke="currentColor" fill="none" width="96" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="28" text-anchor="middle" dominant-baseline="middle">Default</text> <text fill="currentColor" x="48" y="46" text-anchor="middle" dominant-baseline="middle">one image</text></g> <g transform="translate(72, 258)"><rect stroke="currentColor" fill="none" width="96" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="29" text-anchor="middle" dominant-baseline="middle">Runner 1</text></g> <g transform="translate(196, 258)"><rect stroke="currentColor" fill="none" width="96" height="56" rx="8" ry="8"></rect><text fill="currentColor" x="48" y="29" text-anchor="middle" dominant-baseline="middle">Runner 2</text></g></svg>

Every runner in a pool runs the same classes.

Effect Cluster needs this: it gives each shard to any runner in a group, whatever classes the runner has, so every runner must have them all.

A class’s `main` is how the pool finds it. The pool’s program imports the default export of each class’s `main`: the class itself when it’s declared with its implementation, or the Layer `.make` returned when it’s [declared on its own](entities.md#keep-its-code-out-of-its-callers). So `main` is almost always `import.meta.url`, in a file whose default export is the class or the Layer.

Because the pool bundles every class it hosts, a class calling another on the same pool adds nothing to it. Only callers outside the pool, or on another pool, bundle code they never run.

## Split work across pools

A second pool is its own program, image and service, scaled and deployed on its own:

```typescript
import * as ClusterDocker from "@alchemy.run/effect-cluster/Docker";
import * as Layer from "effect/Layer";

export const Database = ClusterDocker.Database("Cluster");

export const Default = ClusterDocker.SwarmPool("Default", { replicas: 2 }).pipe(
  Layer.provide(Database),
);

export const Billing = ClusterDocker.SwarmPool("Billing", { replicas: 4 }).pipe(
  Layer.provide(Database),
);
```

Each pool is its own **shard group**, so its classes’ instances only run on its runners. Pools on the same database are one cluster: an entity on `Default` calls one on `Billing` directly.

## Deploying with Layers

Provide a pool to an implementation’s Effect, whether the class is declared with its implementation or on its own, with whatever its handlers need from them, like RPC middleware:

```typescript
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import * as Ref from "effect/Ref";
import { AuthLive } from "./Auth.ts";
import { Default } from "./Cluster.ts";
import { Room } from "./Room.ts";

export default Room.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    return Effect.gen(function* () {
      const members = yield* Ref.make(new Set<string>());
      return Room.of({
        Join: () => Effect.map(Ref.get(members), (all) => all.size),
        Members: () => Effect.map(Ref.get(members), (all) => [...all]),
      });
    });
  }).pipe(Effect.provide(Default.pipe(Layer.provide(AuthLive)))),
);
```

Or provide it to the Layer `.make` returns, as Effect does with `toLayer`:

```typescript
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { Default } from "./Cluster.ts";
import { Sweeper } from "./Sweeper.ts";

export default Sweeper.make(
  { main: import.meta.url },
  Effect.gen(function* () {
    return Effect.logInfo("sweeping");
  }),
).pipe(Layer.provide(Default));
```

## The gateway

Every pool serves its classes over HTTP to callers outside the cluster, behind a token. Where the gateway is reachable depends on the cloud:

| Cloud | Gateway |
| --- | --- |
| Docker | the service’s name, on the swarm’s network |
| AWS | an internet-facing Application Load Balancer |
| GCP | a regional load balancer on a static address |
| Kubernetes | a Service, in the cluster (or a load balancer, with `serviceType`) |
| Fly | `https://{name}.fly.dev` |
| Hetzner | a Load Balancer |
| Cloudflare | the pool’s Worker, through a service binding from other Workers |

Load balancers check the gateway at `/health`, which needs no token.

## What happens on a deploy

When the host replaces runners, each new runner registers and takes shards as the old ones release them. Instances on a moving shard stop and start again on their new runner, with fresh state. Persisted messages and workflows continue; a call in flight to a moving instance can fail with `EntityNotAssignedToRunner` and is safe to retry.
