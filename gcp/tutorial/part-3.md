---
url: https://alchemy.run/gcp/tutorial/part-3
title: "Part 3: Testing"
description: "Write integration tests that deploy your stack and make HTTP requests against the live Cloud Run service."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

In [Part 2](part-2.md) you deployed a Cloud Run service bound to a Bucket. Now you write tests that deploy the Stack, call the live service, and check it works.

## Install the test harness

`Test.make` ships in `@alchemy.run/test`. Add it as a dev dependency:

```sh
pnpm add -D @alchemy.run/test@latest
```

## Create the test file

`Test.make` takes the same providers and state as your Stack:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Test from "@alchemy.run/test/Bun";
import { expect } from "bun:test";
import * as Effect from "effect/Effect";
import Stack from "../alchemy.run.ts";

const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: GCP.providers(),
  state: GCP.state(),
});
```

## Deploy before the tests

Deploy the Stack once, before any test runs:

```typescript
const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: GCP.providers(),
  state: GCP.state(),
});

const stack = beforeAll(deploy(Stack), { timeout: 600_000 });
```

`beforeAll` returns a handle. `yield*` it inside a test to get the Stack outputs. The first deploy builds an image, so give it a long timeout.

## Read the outputs

```typescript
const stack = beforeAll(deploy(Stack), { timeout: 600_000 });

test(
  "stack exposes a service url",
  Effect.gen(function* () {
    const { url } = yield* stack;

    expect(url).toStartWith("https://");
  }),
);
```

## Run the tests

```sh
bun test test/integ.test.ts
```

The first run deploys. Later runs reuse the deployment and only change what you changed.

## Call the service

Replace the first test with a round trip through the Bucket:

```typescript
import * as Alchemy from "alchemy";
import * as GCP from "@alchemy.run/gcp";
import * as Test from "@alchemy.run/test/Bun";
import { expect } from "bun:test";
import * as Effect from "effect/Effect";
import * as HttpBody from "effect/http/HttpBody";
import * as HttpClient from "effect/http/HttpClient";
import * as Schedule from "effect/Schedule";
import Stack from "../alchemy.run.ts";

const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: GCP.providers(),
  state: GCP.state(),
});

const stack = beforeAll(deploy(Stack), { timeout: 600_000 });

test(
  "stack exposes a service url",
  Effect.gen(function* () {
    const { url } = yield* stack;

    expect(url).toStartWith("https://");
  }),
);
test(
  "PUT and GET round-trip an object",
  Effect.gen(function* () {
    const { url } = yield* stack;

    const put = yield* HttpClient.put(\`${url}/hello.txt\`, {
      body: HttpBody.text("Hello, World!"),
    }).pipe(
      Effect.retry({ schedule: Schedule.exponential("500 millis"), times: 10 }),
    );
    expect(put.status).toBe(204);

    const get = yield* HttpClient.get(\`${url}/hello.txt\`);
    expect(yield* get.text).toBe("Hello, World!");
  }),
  { timeout: 120_000 },
);

test(
  "GET a missing key returns 404",
  Effect.gen(function* () {
    const { url } = yield* stack;
    const response = yield* HttpClient.get(\`${url}/no-such-key\`);
    expect(response.status).toBe(404);
  }),
);
```

`HttpClient` comes with the test harness.

## Let destroy empty the Bucket

The PUT test leaves `hello.txt` in the Bucket. Cloud Storage won’t delete a Bucket that still has objects, so destroy would fail. This Bucket only holds test data, so let destroy empty it first:

```typescript
Effect.gen(function* () {
  const bucket = yield* GCP.Storage.Bucket("Bucket", {});
  const bucket = yield* GCP.Storage.Bucket("Bucket", {
    forceDestroy: true,
  });
  const files = yield* GCP.Storage.ReadWriteBucket(bucket);
```

Leave `forceDestroy` off for Buckets that hold data you need to keep.

## Destroy after tests in CI

Locally, keep the Stack deployed between runs so tests start fast. In CI, tear it down:

```typescript
const stack = beforeAll(deploy(Stack), { timeout: 600_000 });

afterAll.skipIf(!process.env.CI)(destroy(Stack), { timeout: 600_000 });
```

With `CI=true` the Stack is destroyed after the tests. Without it, `skipIf` skips the destroy.

## Recap

You now have:

- `Test.make({ providers, state })` wiring GCP into the test runner
- `beforeAll(deploy(Stack))` deploying once before the tests
- HTTP assertions against the live Cloud Run URL
- `forceDestroy: true` so destroy can delete a Bucket tests wrote to
- `afterAll.skipIf(!process.env.CI)(destroy(Stack))` cleaning up in CI

In [Part 4](part-4.md), you give every developer, environment, and pull request its own copy of the Stack.
