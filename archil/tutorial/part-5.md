---
url: https://alchemy.run/archil/tutorial/part-5
title: "Part 5: Testing"
description: "Deploy the Stack in a test, fork the set-up sandbox, and check what the fork contains."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Install the test harness

```sh
pnpm add -D @alchemy.run/test@latest
```

## Deploy once per test file

`Test.make` takes the same providers and state as your Stack:

```typescript
import { expect } from "bun:test";
import * as Archil from "@alchemy.run/archil";
import * as Test from "@alchemy.run/test/Bun";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as HttpClient from "effect/http/HttpClient";
import * as HttpClientRequest from "effect/http/HttpClientRequest";
import * as Schedule from "effect/Schedule";
import Stack from "../alchemy.run.ts";

const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: Archil.providers(),
  state: Alchemy.localState(),
  profile: process.env.ALCHEMY_PROFILE,
});

const stack = beforeAll(deploy(Stack), { timeout: 300_000 });
afterAll.skipIf(!!process.env.NO_DESTROY)(destroy(Stack), { timeout: 180_000 });
```

The test deploys the Stack to a stage of its own, `Golden` included, and destroys it afterwards.

## Retry while the program starts

A new program sandbox can take a few seconds to answer:

```typescript
const run = (url: string, command: string) =>
  Effect.gen(function* () {
    const client = yield* HttpClient.HttpClient;
    const res = yield* client.execute(HttpClientRequest.post(\`${url}/run\`).pipe(HttpClientRequest.bodyText(command))).pipe(
      Effect.flatMap((res) => (res.status >= 500 ? Effect.fail(new Error(\`HTTP ${res.status}\`)) : Effect.succeed(res))),
      Effect.retry({ schedule: Schedule.spaced("2 seconds"), times: 20 }),
    );
    return (yield* res.json) as { stdout: string; exitCode: number };
  });
```

## Check a fork starts from Golden

```typescript
test(
  "forks Golden for each request",
  Effect.gen(function* () {
    const { url } = yield* stack;
    const both = yield* run(url, "ls ..");
    expect(both.stdout).toContain("slugify");
  }),
  { timeout: 300_000 },
);
```

`slugify` is only there if the fork came from the new `Golden`. The program deletes the fork when the request ends, so none is left behind.

## Run it

```sh
bun test
```

Set `NO_DESTROY=1` to keep the deployment between runs while you iterate.

## Next

[Part 6](part-6.md) deploys more copies, from CI.
