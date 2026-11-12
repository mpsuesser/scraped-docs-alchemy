---
url: https://alchemy.run/gcp
title: "GCP"
description: "Build on Google Cloud with Alchemy. Declare Cloud Run services, jobs, Firestore, Cloud Storage and Pub/Sub in TypeScript, and let bindings grant each runtime's service account the roles its code uses."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Google Cloud is a cloud platform with services for compute, storage, databases, networking, data analytics and AI, in regions around the world. It’s strongest at data and machine learning, with BigQuery, Spanner, and Vertex AI with Gemini, and at Kubernetes, with GKE.

**New to Google Cloud?** Start with [What is GCP?](gcp/what-is-gcp.md).

## A quick example

A Firestore database is a declaration:

```typescript
import * as GCP from "@alchemy.run/gcp";

export const Links = GCP.Firestore.Database("Links", { type: "FIRESTORE_NATIVE" });
```

A Cloud Run service binds it, and reads and writes documents as plain objects. The binding grants the service’s account access to that one database:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Links } from "./links.ts";

export default class Api extends GCP.Function<Api>()(
  "Api",
  { main: import.meta.url, invokerIamDisabled: true },
  Effect.gen(function* () {
    const db = yield* GCP.Firestore.ReadWriteDatabase(Links);
    return {
      fetch: Effect.gen(function* () {
        yield* db.set("links/demo", { url: "https://alchemy.run" });
        const link = yield* db.get("links/demo");
        return yield* HttpServerResponse.json(link?.fields ?? {});
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Firestore.ReadWriteDatabaseHttp)),
) {}
```

And the Stack deploys it to the project your profile names:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: GCP.providers(), state: GCP.state() },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.uri };
  }),
);
```

## Where to start

**Want to build?** The [tutorial](gcp/tutorial/part-1.md) goes from an empty folder to a tested service deployed from CI, one step at a time. Start with [Setup](gcp/setup.md) to connect your project.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Networking

- [VPC & private networking](gcp/networking/vpc.md)
- [Private services](gcp/networking/private-services.md)

### Observability

- [Logging & Monitoring](gcp/observability/monitoring.md)

### AI

- [Vertex AI](gcp/ai/vertex-ai.md)
