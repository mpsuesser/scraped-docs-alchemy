---
url: https://alchemy.run/gcp/compute/cloud-run
title: "Cloud Run"
description: "Run an Effect HTTP service on Cloud Run with GCP.Function — public or private URLs, environment, scaling, and bindings that grant IAM."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

`GCP.Function` runs an Effect program as a Cloud Run service. You write a `fetch` handler, Alchemy builds a container image, pushes it to Artifact Registry, and deploys a new revision. The service scales with traffic and down to zero.

## Instances and revisions

Each deploy creates a **revision**, and traffic moves to it. A revision runs in **instances**: copies of your container that each take up to 80 requests at once by default. Cloud Run starts more instances as traffic grows and stops them when it falls, down to none.

<svg viewBox="0 0 670 214" style="max-width: 770px" role="img" aria-label="Each instance serves many requests side by side. When traffic grows past what the running instances take, Cloud Run starts another, and when it stops, it stops them all."><line stroke-opacity="0.2" stroke="currentColor" x1="221.20000000000002" y1="24" x2="221.20000000000002" y2="208"></line><line stroke-opacity="0.2" stroke="currentColor" x1="490" y1="24" x2="490" y2="208"></line><g><rect stroke="currentColor" fill="none" x="90" y="33" width="576" height="98" rx="8"></rect><text fill="currentColor" x="4" y="86">instance 1</text> <g><rect stroke="currentColor" fill="none" x="98" y="40" width="33.599999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="131.6" y="40" width="100.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="182" y="56" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="165.2" y="70" width="112.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="221.20000000000002" y="86" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="187.60000000000002" y="100" width="123.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="249.20000000000002" y="116" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="288.4" y="40" width="123.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="350" y="56" text-anchor="middle">request</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="134" x2="670" y2="134"></line><rect stroke="currentColor" fill="none" x="90" y="137" width="576" height="68" rx="8"></rect><text fill="currentColor" x="4" y="175">instance 2</text> <g><rect stroke="currentColor" fill="none" x="221.20000000000002" y="144" width="33.599999999999966" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="254.79999999999998" y="144" width="112.00000000000003" height="24" rx="5"></rect><text fill="currentColor" x="310.8" y="160" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="266" y="174" width="123.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="327.6" y="190" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="221.20000000000002" y="18" text-anchor="middle">traffic grows</text> <text fill="currentColor" x="490" y="18" text-anchor="middle">idle: no instances</text></svg> <svg viewBox="0 0 330 234" style="max-width: 379px" role="img" aria-label="Each instance serves many requests side by side. When traffic grows past what the running instances take, Cloud Run starts another, and when it stops, it stops them all."><line stroke-opacity="0.2" stroke="currentColor" x1="69.96000000000001" y1="24" x2="69.96000000000001" y2="228"></line><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="24" x2="222.6" y2="228"></line><g><rect stroke="currentColor" fill="none" x="0" y="51" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="44">instance 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="19.08" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="19.08" y="54" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="70" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="38.16" y="84" width="63.60000000000001" height="24" rx="5"></rect><text fill="currentColor" x="69.96000000000001" y="100" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="50.88" y="114" width="69.96000000000001" height="24" rx="5"></rect><text fill="currentColor" x="85.86000000000001" y="130" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="54" width="69.96" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="70" text-anchor="middle">request</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="144" x2="330" y2="144"></line><rect stroke="currentColor" fill="none" x="0" y="165" width="326" height="60" rx="8"></rect><text fill="currentColor" x="0" y="158">instance 2</text> <g><rect stroke="currentColor" fill="none" x="69.96000000000001" y="168" width="19.079999999999984" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="89.03999999999999" y="168" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="120.83999999999999" y="184" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="198" width="69.96000000000002" height="24" rx="5"></rect><text fill="currentColor" x="130.38" y="214" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="69.96000000000001" y="18" text-anchor="middle">traffic grows</text> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">idle: no instances</text></svg>

requestcold start

Each instance serves many requests side by side. When traffic grows past what the running instances take, Cloud Run starts another, and when it stops, it stops them all.

You pay while instances serve requests, so an idle service costs nothing. Set `scaling.minInstanceCount` to keep some warm and skip the cold start.

## Declare a service

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends GCP.Function<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("Hello from Cloud Run!")),
    };
  }),
) {}
```

`GCP.Function` is the same class as `GCP.Run.Service`. `main` names the file to bundle.

## Add it to the Stack

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import Api from "./src/Api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.uri };
  }),
);
```

`api.uri` is the service’s `https://….run.app` URL.

## Make it public

A new service only accepts callers with a Google identity token and `roles/run.invoker`. Set `invokerIamDisabled` to let anyone call it:

```typescript
export default class Api extends GCP.Function<Api>()(
  "Api",
  { main: import.meta.url },
  { main: import.meta.url, invokerIamDisabled: true },
```

Leave it off for internal services. Another service calls a private one with [`InvokeService`](../networking/private-services.md).

## Read the request

`fetch` is an Effect. Get the request from `HttpServerRequest`:

```typescript
import { HttpServerRequest } from "effect/http/HttpServerRequest";

fetch: Effect.gen(function* () {
  const request = yield* HttpServerRequest;
  const url = new URL(request.originalUrl, "http://localhost");

  if (request.method === "POST" && url.pathname === "/echo") {
    const body = yield* request.json;
    return yield* HttpServerResponse.json(body);
  }

  return HttpServerResponse.text("Not found", { status: 404 });
}).pipe(Effect.orDie),
```

For typed routes, use [Effect HTTP](../apis/effect-http-api.md) or [Effect RPC](../apis/effect-rpc.md). Both produce a `fetch` handler.

## Bind resources

Declare a resource in the constructor and bind it. The binding returns a client and grants the service the role it needs on that resource:

```typescript
Effect.gen(function* () {
  const links = yield* GCP.Firestore.Database("Links", {
    type: "FIRESTORE_NATIVE",
  });
  const db = yield* GCP.Firestore.ReadWriteDatabase(links);

  return {
    fetch: Effect.gen(function* () {
      const doc = yield* db.get("links/home");
      return yield* HttpServerResponse.json(doc?.fields ?? {});
    }).pipe(Effect.orDie),
  };
}).pipe(Effect.provide(GCP.Firestore.ReadWriteDatabaseHttp)),
```

Each service gets its own service account. Bindings grant roles to it and nothing else. See [How bindings grant IAM](../security/iam.md).

## Pass environment variables

`env` sets plain container environment variables:

```typescript
{
  main: import.meta.url,
  env: {
    LOG_LEVEL: "debug",
    API_BASE: other.uri,
  },
}
```

Values can be Outputs from other resources. For credentials, use [Secret Manager](../security/secrets-kms.md) instead.

## Choose the region

A service deploys to the Stack’s region (your profile’s region, else `us-central1`). Pin it with `location`:

```typescript
{ main: import.meta.url, location: "europe-west1" }
```

Changing `location` replaces the service.

## Tune scaling

Keep warm instances to avoid cold starts, or cap the maximum:

```typescript
{
  main: import.meta.url,
  scaling: { minInstanceCount: 1, maxInstanceCount: 20 },
}
```

## Size the container

Set CPU, memory, timeout, and concurrency on the revision template:

```typescript
{
  main: import.meta.url,
  template: {
    timeout: "60s",
    maxInstanceRequestConcurrency: 40,
    containers: [{ resources: { limits: { cpu: "2", memory: "1Gi" } } }],
  },
}
```

Every template change rolls out a new revision.

## Cache build layers

Image builds reuse the local BuildKit cache of the machine they run on. A fresh CI runner has none and rebuilds every step. Keep the layers in Artifact Registry with `build.cache`:

```typescript
{
  main: import.meta.url,
  build: { cache: true },
}
```

Builds then import and export every stage’s layers at the repository’s `:buildcache` tag. The cache is off by default and never changes the image, so turning it on doesn’t roll out a revision. Other backends take BuildKit’s cache backends, as in [Docker builds](../../docker/build-caches.md).

## Check health

Cloud Run’s default startup probe passes as soon as the port accepts a connection. Give it an HTTP probe so a revision only takes traffic once the program can serve:

```typescript
{
  main: import.meta.url,
  template: {
    containers: [
      {
        startupProbe: { httpGet: { path: "/health" }, periodSeconds: 2, failureThreshold: 30 },
        livenessProbe: { httpGet: { path: "/health" }, periodSeconds: 10 },
      },
    ],
  },
}
```

A failing liveness probe restarts the container. Probes can also use `tcpSocket` or `grpc`. Remove `startupProbe` to go back to the default.

## Add a sidecar

Containers after the first run next to `main` and reach it on `localhost`. Only the first container takes requests:

```typescript
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
}
```

`dependsOn` starts `main` after `cache` passes its startup probe. [Sidecars](sidecars.md) covers start order, running your own program as a sidecar, and binding it for a typed client.

## Receive events

Event sources make Google call your service. Each one creates the delivery (a push subscription, a scheduler job, an Eventarc trigger) and lets only that delivery in, so the service can stay private:

```typescript
yield* GCP.PubSub.consumeTopicMessages(Orders, (messages) =>
  messages.pipe(Stream.runForEach(({ message }) => handle(message))),
);
// …provided with Effect.provide(GCP.Run.TopicEventSource)
```

See [Pub/Sub](../messaging/pubsub.md), [Cloud Scheduler](../messaging/scheduler.md), [Storage events](../messaging/storage-events.md), and [Eventarc](../messaging/eventarc.md).

## Run without main

Leave out `main` to deploy any container image:

```typescript
const hello = yield* GCP.Run.Service("Hello", {
  invokerIamDisabled: true,
  template: {
    containers: [{ image: "us-docker.pkg.dev/cloudrun/container/hello" }],
  },
});
```

An image service can list sidecars too; see [Sidecars](sidecars.md).

## Destroy

`alchemy destroy` deletes the service, its image repository, and the service account Alchemy created for it, and revokes that account’s roles.

## Example

[`examples/gcp-cloud-run-api`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-cloud-run-api) is a link shortener on one service: Firestore for links, Secret Manager for the API key.

## Where next

- [Firestore](../data/firestore.md) — the datastore most services start with.
- [Cloud Run jobs](jobs.md) — work that runs to completion.
- [Run reference](https://alchemy.run/providers/gcp/run#service) — every prop and attribute.
