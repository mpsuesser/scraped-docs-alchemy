---
url: https://alchemy.run/daytona/tutorial/part-5
title: "Part 5: Testing"
description: "Deploy the Stack in a test, start sandboxes from the warm snapshot, and check what they contain."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Deploy once per test file

```typescript
import { expect } from "bun:test";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Daytona from "@alchemy.run/daytona";
import * as Test from "@alchemy.run/test/Bun";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as HttpClientRequest from "effect/http/HttpClientRequest";
import * as Layer from "effect/Layer";
import Stack from "../alchemy.run.ts";

const { test, beforeAll, afterAll, deploy, destroy } = Test.make({
  providers: Layer.mergeAll(Cloudflare.providers(), Daytona.providers()),
  state: Alchemy.localState(),
});

const stack = beforeAll(deploy(Stack));
afterAll.skipIf(!!process.env.NO_DESTROY)(destroy(Stack));
```

The test deploys the Stack to a stage of its own, snapshots included, and destroys it afterwards.

## Check a sandbox starts from Warm

```typescript
test(
  "starts each sandbox from the Warm snapshot",
  Effect.gen(function* () {
    const { url } = yield* stack;
    const res = yield* Test.executeWhenReady(
      HttpClientRequest.post(\`${url}/run\`).pipe(HttpClientRequest.bodyText("node --version")),
    );
    const body = (yield* res.json) as { exitCode: number; result: string };
    expect(body.result).toMatch(/^v24\./);
  }),
  { timeout: 300_000 },
);
```

Node.js 24 is only there if the sandbox booted from the new `Warm`. The Worker deletes the sandbox when the request ends, so none is left behind.

## Run it

```sh
bun test
```

Set `NO_DESTROY=1` to keep the deployment, and the snapshots, between runs while you iterate.

## Next

[Part 6](part-6.md) deploys more copies, from CI.
