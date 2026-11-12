---
url: https://alchemy.run/gcp/compute/sidecars
title: "Cloud Run sidecars"
description: "Run more containers next to a Cloud Run service — plain images, GCP.Run.Container resources, or your own Effect programs — and control the order they start in."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Cloud Run instance can run several containers. The first one takes requests. The rest are sidecars, and every container in the instance reaches the others on `localhost`.

instanceone copy of the revision

apptakes requests ·:8080

cacheredis · localhost:6379

collectorlocalhost:4317

Every container in an instance shares its network. The first one takes the requests and reaches its sidecars on localhost.

A sidecar can be a plain image you list in the template, a `GCP.Run.Container` that runs an image, or a `GCP.Run.Container` that runs your own Effect program.

## Add a sidecar

List more containers after the first in `template.containers`:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const proxy = yield* GCP.Run.Service("Proxy", {
      template: {
        containers: [
          {
            image: "us-docker.pkg.dev/cloudrun/container/hello",
            ports: [{ containerPort: 8080 }],
          },
          { name: "cache", image: "docker.io/library/redis:7-alpine" },
        ],
      },
    });
    return { url: proxy.uri };
  }),
);
```

Only one container may have `ports`, and that container takes the requests.

## Start containers in order

Cloud Run starts every container at once, and the instance takes requests as soon as the first one passes its startup probe. The first requests can reach a sidecar before it listens. `dependsOn` holds a container back until the containers it names pass their startup probes:

```typescript
const proxy = yield* GCP.Run.Service("Proxy", {
  template: {
    containers: [
      {
        image: "us-docker.pkg.dev/cloudrun/container/hello",
        ports: [{ containerPort: 8080 }],
        dependsOn: ["cache"],
      },
      { name: "cache", image: "docker.io/library/redis:7-alpine" },
      {
        name: "cache",
        image: "docker.io/library/redis:7-alpine",
        startupProbe: { tcpSocket: { port: 6379 } },
      },
    ],
  },
});
```

Give each dependency a startup probe that checks it can serve. A sidecar has no `ports`, so its probe names the port.

<svg viewBox="0 0 698.4 124" style="max-width: 803px" role="img" aria-label="The app doesn't start until the cache passes its startup probe, so the first request can't arrive before the cache listens."><line stroke-opacity="0.2" stroke="currentColor" x1="260.8" y1="24" x2="260.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="473.6" y1="24" x2="473.6" y2="118"></line><g><text fill="currentColor" x="4" y="56">cache</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="40" width="134.4" height="24" rx="5"></rect><text fill="currentColor" x="193.60000000000002" y="56" text-anchor="middle">starting</text></g> <g><rect stroke="currentColor" fill="none" x="260.8" y="40" width="425.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="473.6" y="56" text-anchor="middle">serving on:6379</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="698.4" y2="74"></line><text fill="currentColor" x="4" y="94">app</text> <text fill="currentColor" x="4" y="108">dependsOn: cache</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="84" width="134.4" height="24" rx="5"></rect><text fill="currentColor" x="193.60000000000002" y="100" text-anchor="middle">waiting</text></g> <g><rect stroke="currentColor" fill="none" x="260.8" y="84" width="212.8" height="24" rx="5"></rect><text fill="currentColor" x="367.20000000000005" y="100" text-anchor="middle">starting</text></g> <g><rect stroke="currentColor" fill="none" x="473.6" y="84" width="212.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="580" y="100" text-anchor="middle">taking requests</text></g></g> <text fill="currentColor" x="260.8" y="18" text-anchor="middle">cache passes its probe</text> <text fill="currentColor" x="473.6" y="18" text-anchor="middle">instance takes requests</text></svg> <svg viewBox="0 0 330 158" style="max-width: 379px" role="img" aria-label="The app doesn't start until the cache passes its startup probe, so the first request can't arrive before the cache listens."><line stroke-opacity="0.2" stroke="currentColor" x1="76.32" y1="38" x2="76.32" y2="152"></line><line stroke-opacity="0.2" stroke="currentColor" x1="197.16" y1="38" x2="197.16" y2="152"></line><g><text fill="currentColor" x="0" y="58">cache</text> <g><rect stroke="currentColor" fill="none" x="0" y="68" width="76.32" height="24" rx="5"></rect><text fill="currentColor" x="38.16" y="84" text-anchor="middle">starting</text></g> <g><rect stroke="currentColor" fill="none" x="76.32" y="68" width="241.68" height="24" rx="5"></rect><text fill="currentColor" x="197.16" y="84" text-anchor="middle">serving on:6379</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="98" x2="330" y2="98"></line><text fill="currentColor" x="0" y="112">app <tspan fill="currentColor" dx="8">dependsOn: cache</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="122" width="76.32" height="24" rx="5"></rect><text fill="currentColor" x="38.16" y="138" text-anchor="middle">waiting</text></g> <g><rect stroke="currentColor" fill="none" x="76.32" y="122" width="120.84" height="24" rx="5"></rect><text fill="currentColor" x="136.74" y="138" text-anchor="middle">starting</text></g> <g><rect stroke="currentColor" fill="none" x="197.16" y="122" width="120.84" height="24" rx="5"></rect><text fill="currentColor" x="257.58" y="138" text-anchor="middle">taking requests</text></g></g> <text fill="currentColor" x="76.32" y="18" text-anchor="middle">cache passes its probe</text> <text fill="currentColor" x="197.16" y="32" text-anchor="middle">instance takes requests</text></svg>

The app doesn't start until the cache passes its startup probe, so the first request can't arrive before the cache listens.

## Chain dependencies

A container that others wait on can wait too. Names chain into an order, and containers that don’t depend on each other still start together:

```text
cache ✓ → app ✓ → proxy ✓ → requests
```

If a dependency never passes its startup probe, the containers waiting on it never start. The new revision fails and traffic stays on the previous one.

`dependsOn` only orders startup. A dependency that crashes later doesn’t stop the containers that wait on it. Give it a `livenessProbe` so Cloud Run restarts it.

## Declare a sidecar as a Container

`GCP.Run.Container` declares a sidecar as a resource. List it in `containers` like a plain spec:

```typescript
const cache = yield* GCP.Run.Container("Cache", {
  image: "docker.io/library/redis:7-alpine",
  port: 6379,
  startupProbe: { tcpSocket: {} },
});
const proxy = yield* GCP.Run.Service("Proxy", {
  template: {
    containers: [
      {
        image: "us-docker.pkg.dev/cloudrun/container/hello",
        ports: [{ containerPort: 8080 }],
        dependsOn: ["cache"],
        dependsOn: [cache.name],
      },
      {
        name: "cache",
        image: "docker.io/library/redis:7-alpine",
        startupProbe: { tcpSocket: { port: 6379 } },
      },
      cache,
    ],
  },
});
```

`cache.name` is an Output. Alchemy names the container from its id (or its `name` prop) and puts that name into `dependsOn`, so you never write it twice. `port` fills in the port the probe checks.

## Wait for another Container

A Container’s own `dependsOn` takes another Container’s `name` the same way:

```typescript
const web = yield* GCP.Run.Container("Web", {
  image: "docker.io/library/nginx:1.27-alpine",
  port: 80,
  startupProbe: { tcpSocket: {} },
});
const cache = yield* GCP.Run.Container("Cache", {
  image: "docker.io/library/redis:7-alpine",
  port: 6379,
  startupProbe: { tcpSocket: {} },
  dependsOn: [web.name],
});
const proxy = yield* GCP.Run.Service("Proxy", {
  template: {
    containers: [
      {
        image: "us-docker.pkg.dev/cloudrun/container/hello",
        ports: [{ containerPort: 8080 }],
        dependsOn: [cache.name],
      },
      cache,
      web,
    ],
  },
});
```

```text
web ✓ → cache ✓ → proxy ✓ → requests
```

Every name in `dependsOn` has to belong to a container in the same `containers` list.

## Run your own program in a Container

Give a Container `main` and an Effect, and Alchemy bundles your program into its own image. Return methods next to `fetch`:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Worker extends GCP.Run.Container<Worker>()(
  "Worker",
  { main: import.meta.url, port: 3000, startupProbe: { httpGet: { path: "/health" } } },
  Effect.gen(function* () {
    return {
      greet: (name: string) => Effect.succeed(\`hello ${name}\`),
      fetch: Effect.succeed(HttpServerResponse.text("ok")),
    };
  }),
) {}
```

An image service lists it like any other Container:

```typescript
import Worker from "./src/worker.ts";

const worker = yield* Worker;
const proxy = yield* GCP.Run.Service("Proxy", {
  template: {
    containers: [
      {
        image: "us-docker.pkg.dev/cloudrun/container/hello",
        ports: [{ containerPort: 8080 }],
        dependsOn: [cache.name],
        dependsOn: [worker.name],
      },
      cache,
      web,
      worker,
    ],
  },
});
```

The proxy reaches it at `http://localhost:3000`.

## Add sidecars to an Effect service

A service with `main` runs your program in the first container. Leave that entry empty and list sidecars after it:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends GCP.Run.Service<Api>()(
  "Api",
  {
    main: import.meta.url,
    template: {
      containers: [
        { dependsOn: ["cache"] },
        {
          name: "cache",
          image: "docker.io/library/redis:7-alpine",
          startupProbe: { tcpSocket: { port: 6379 } },
        },
      ],
    },
  },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("hello")),
    };
  }),
) {}
```

## Bind a Container

An Effect service binds a Container instead of listing it. `GCP.Run.bindContainer` adds the Container to the revision and to the service’s `dependsOn`, and returns a client that calls it on `localhost`:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Worker from "./worker.ts";

export default class Api extends GCP.Run.Service<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const worker = yield* GCP.Run.bindContainer(Worker);
    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text(yield* worker.greet("sam"));
      }),
    };
  }),
) {}
```

`worker.greet` calls the method over HTTP, and `worker.fetch` sends a request to the Container’s `fetch`. For typed clients, bind with an [Effect RPC](../apis/effect-rpc.md) schema or an [Effect HTTP API](../apis/effect-http-api.md).

## Bind a sidecar’s port

A sidecar that doesn’t speak HTTP, like Redis, is bound by port. Declare it in its own module:

```typescript
import * as GCP from "@alchemy.run/gcp";

export const Cache = GCP.Run.Container("Cache", {
  image: "docker.io/library/redis:7-alpine",
  port: 6379,
  startupProbe: { tcpSocket: {} },
});
```

`GCP.Run.bindPort` adds it to the revision and the `dependsOn` the same way, and returns its address:

```typescript
import { Cache } from "./cache.ts";

Effect.gen(function* () {
  const cache = yield* GCP.Run.bindPort(Cache, 6379);
  // cache.url === "http://localhost:6379"
  return {
    fetch: Effect.succeed(HttpServerResponse.text("hello")),
  };
}),
```

## Bind a Container from a Container

A Container’s program binds sidecars the same way a service does. Here `Worker` binds `Cache`:

```typescript
import { Cache } from "./cache.ts";

export default class Worker extends GCP.Run.Container<Worker>()(
  "Worker",
  { main: import.meta.url, port: 3000, startupProbe: { httpGet: { path: "/health" } } },
  Effect.gen(function* () {
    const cache = yield* GCP.Run.bindPort(Cache, 6379);
    return {
      greet: (name: string) => Effect.succeed(\`hello ${name}\`),
      fetch: Effect.succeed(HttpServerResponse.text("ok")),
    };
  }),
) {}
```

`Cache` joins whichever revision `Worker` runs in, and `Worker` waits for it. That holds whether a service binds `Worker` or lists it:

```text
cache ✓ → worker ✓ → api ✓ → requests
```

Other bindings in a Container work as they do in a service: their environment variables land on the Container, and their roles go to the service’s account. A Container’s own `dependsOn` is kept alongside the names its bindings add.

## List or bind

A service that runs an image lists its sidecars, and names what each one waits for with `dependsOn`. A service or Container that runs your program binds its sidecars, which gives it a client and orders startup for it.
