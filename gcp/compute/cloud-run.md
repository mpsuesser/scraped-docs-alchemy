---
url: https://alchemy.run/gcp/compute/cloud-run
title: "Cloud Run"
description: "Run an Effect HTTP service on Cloud Run with GCP.Function — public or private URLs, environment, scaling, and bindings that grant IAM."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

`GCP.Function` runs an Effect program as a Cloud Run service. You
write a `fetch` handler, Alchemy builds a container image, pushes it
to Artifact Registry, and deploys a new revision. The service scales
with traffic and down to zero.

:::note
The image is built on your machine, so Docker has to be running when
you deploy. To deploy without Docker, use
[Cloud Functions](cloud-functions.md).
:::

## Declare a service

```typescript
// src/Api.ts
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

`GCP.Function` is the same class as `GCP.Run.Service`. `main` names
the file to bundle.

## Add it to the Stack

```typescript
// alchemy.run.ts
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

`api.uri` is the service's `https://….run.app` URL.

## Make it public

A new service only accepts callers with a Google identity token and
`roles/run.invoker`. Set `invokerIamDisabled` to let anyone call it:

```diff lang="typescript"
export default class Api extends GCP.Function<Api>()(
  "Api",
-  { main: import.meta.url },
+  { main: import.meta.url, invokerIamDisabled: true },
```

Leave it off for internal services. Another service calls a private
one with [`InvokeService`](../networking/private-services.md).

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

For typed routes, use [Effect HTTP](../apis/effect-http-api.md) or
[Effect RPC](../apis/effect-rpc.md). Both produce a `fetch` handler.

## Bind resources

Declare a resource in the constructor and bind it. The binding returns
a client and grants the service the role it needs on that resource:

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

Each service gets its own service account. Bindings grant roles to it
and nothing else. See [How bindings grant IAM](../security/iam.md).

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

Values can be Outputs from other resources. For credentials, use
[Secret Manager](../security/secrets-kms.md) instead.

## Choose the region

A service deploys to the Stack's region (your profile's region, else
`us-central1`). Pin it with `location`:

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

## Receive events

Event sources make Google call your service. Each one creates the
delivery (a push subscription, a scheduler job, an Eventarc trigger)
and lets only that delivery in, so the service can stay private:

```typescript
yield* GCP.PubSub.consumeTopicMessages(Orders, (messages) =>
  messages.pipe(Stream.runForEach(({ message }) => handle(message))),
);
// …provided with Effect.provide(GCP.Run.TopicEventSource)
```

See [Pub/Sub](../messaging/pubsub.md),
[Cloud Scheduler](../messaging/scheduler.md),
[Storage events](../messaging/storage-events.md), and
[Eventarc](../messaging/eventarc.md).

## Run without `main`

Leave out `main` to deploy any container image:

```typescript
const hello = yield* GCP.Run.Service("Hello", {
  invokerIamDisabled: true,
  template: {
    containers: [{ image: "us-docker.pkg.dev/cloudrun/container/hello" }],
  },
});
```

## Destroy

`alchemy destroy` deletes the service, its image repository, and the
service account Alchemy created for it, and revokes that account's
roles.

## Example

[`examples/gcp-cloud-run-api`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-cloud-run-api)
is a link shortener on one service: Firestore for links, Secret
Manager for the API key.

## Where next

- [Firestore](../data/firestore.md) — the datastore most services start
  with.
- [Cloud Run jobs](jobs.md) — work that runs to completion.
- [Run reference](https://alchemy.run/providers/gcp/run#service) — every prop and
  attribute.
