---
url: https://alchemy.run/aws/compute/web-functions
title: "Lambda Web Functions"
description: "Run an HTTP server on AWS Lambda Web Functions — an Effect fetch handler or any Node server, with streaming responses, many concurrent requests per environment, and a built-in HTTPS endpoint."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Lambda Web Function runs an HTTP server on Lambda. Where a [Lambda Function](lambda.md) runs one invocation at a time per environment, a Web Function’s environment is a long-running server that takes many requests at once, up to 128. Responses stream to the client as your server writes them, and there’s nothing to run while nobody calls it.

## Environments, endpoints and revisions

Your server runs in **environments**, each a copy of your code with 2 GB of memory and 2 vCPUs. Lambda starts more as traffic grows and stops them as it falls. Because each one serves many requests, a spike starts few environments, and requests share warm connection pools.

<svg viewBox="0 0 704.8 170" style="max-width: 811px" role="img" aria-label="The same spike that started two more environments on the Lambda page fits in the one environment that's already warm. Its requests run side by side and share its connections."><line stroke-opacity="0.2" stroke="currentColor" x1="211.2" y1="24" x2="211.2" y2="164"></line><g><rect stroke="currentColor" fill="none" x="124.80000000000001" y="33" width="576" height="128" rx="8"></rect><text fill="currentColor" x="4" y="95">environment 1</text> <text fill="currentColor" x="4" y="109">up to 128 at once</text> <g><rect stroke="currentColor" fill="none" x="132.8" y="40" width="39.19999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="172" y="40" width="106.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="225.2" y="56" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="211.2" y="70" width="106.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="264.4" y="86" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="216.8" y="100" width="106.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="270" y="116" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="222.40000000000003" y="130" width="106.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="275.6" y="146" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="289.6" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="340" y="56" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="334.4" y="70" width="100.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="384.8" y="86" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="412.8" y="40" width="100.7999999999999" height="24" rx="5"></rect><text fill="currentColor" x="463.19999999999993" y="56" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="547.2" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="597.6" y="56" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="211.2" y="18" text-anchor="middle">traffic spikes</text></svg> <svg viewBox="0 0 330 180" style="max-width: 379px" role="img" aria-label="The same spike that started two more environments on the Lambda page fits in the one environment that's already warm. Its requests run side by side and share its connections."><line stroke-opacity="0.2" stroke="currentColor" x1="44.519999999999996" y1="24" x2="44.519999999999996" y2="174"></line><g><rect stroke="currentColor" fill="none" x="0" y="51" width="326" height="120" rx="8"></rect><text fill="currentColor" x="0" y="44">environment 1 <tspan fill="currentColor" dx="8">up to 128 at once</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="22.259999999999998" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="22.259999999999998" y="54" width="60.42000000000001" height="24" rx="5"></rect><text fill="currentColor" x="52.47" y="70" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="84" width="60.41999999999999" height="24" rx="5"></rect><text fill="currentColor" x="74.72999999999999" y="100" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="47.699999999999996" y="114" width="60.419999999999995" height="24" rx="5"></rect><text fill="currentColor" x="77.91" y="130" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="50.88" y="144" width="60.419999999999995" height="24" rx="5"></rect><text fill="currentColor" x="81.09" y="160" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="89.03999999999999" y="54" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="117.66" y="70" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="114.47999999999999" y="84" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="100" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="54" width="57.23999999999998" height="24" rx="5"></rect><text fill="currentColor" x="187.62" y="70" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="235.32" y="54" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="263.94" y="70" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="44.519999999999996" y="18" text-anchor="middle">traffic spikes</text></svg>

requestcold start

The same spike that started two more environments on the Lambda page fits in the one environment that's already warm. Its requests run side by side and share its connections.

Requests arrive through an **endpoint**: an HTTPS URL with its own auth, rate limit and cap on environments. Every Web Function has one, and you can add more. Each deploy publishes a new **revision**, and the endpoint moves its traffic to it, all at once or a share at a time.

Use a Web Function for anything that answers HTTP: APIs, server-rendered sites, AI responses streamed token by token. The [AWS websites](../frontend/websites.md) run their servers on one. For queues, streams, schedules and other events, use a [Lambda Function](lambda.md).

## Declare a Web Function

Create `src/api.ts` and return a `fetch` handler:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends AWS.Lambda.WebFunction<Api>()(
  "Api",
  { main: import.meta.url, authType: "ApplicationManaged" },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("Hello from Lambda!")),
    };
  }),
) {}
```

Your `fetch` handler runs on a Node HTTP server inside the Web Function, so the same `effect/http` code you’d run anywhere else works unchanged. `authType: "ApplicationManaged"` makes its endpoint public: every request reaches your code.

## Deploy it

Yield the class in your Stack and return its `url`:

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: AWS.providers(),
    state: AWS.state(),
  },
  Effect.gen(function* () {
    const api = yield* Api;
    return { url: api.url };
  }),
);
```

```sh
pnpm alchemy deploy
```

The `url` looks like `https://<id>.<shard>.lambda-web.us-west-2.on.aws`. Every deploy publishes a new revision and moves all traffic to it.

## Stream a response

Return a streamed body and the client receives each chunk as soon as you produce it:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Stream from "effect/Stream";

export default class Api extends AWS.Lambda.WebFunction<Api>()(
  "Api",
  { main: import.meta.url, authType: "ApplicationManaged" },
  Effect.gen(function* () {
    return {
      fetch: Effect.succeed(HttpServerResponse.text("Hello from Lambda!")),
      fetch: Effect.succeed(
        HttpServerResponse.stream(
          Stream.make("Hello", " from", " Lambda!").pipe(
            Stream.tap(() => Effect.sleep("200 millis")),
            Stream.encodeText,
          ),
          { contentType: "text/plain" },
        ),
      ),
    };
  }),
) {}
```

## Use AWS resources

Bindings work the same as on a Lambda Function. Binding an operation grants the function’s role exactly that permission:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Api extends AWS.Lambda.WebFunction<Api>()(
  "Api",
  { main: import.meta.url, authType: "ApplicationManaged" },
  Effect.gen(function* () {
    const bucket = yield* AWS.S3.Bucket("Uploads");
    const putObject = yield* AWS.S3.PutObject(bucket);
    return {
      fetch: Effect.succeed(HttpServerResponse.text("Hello from Lambda!")),
      fetch: Effect.gen(function* () {
        yield* putObject({ Key: "hello.txt", Body: "Hello!" });
        return HttpServerResponse.text("Saved");
      }).pipe(Effect.orDie),
    };
  }),
  }).pipe(Effect.provide(AWS.S3.PutObjectHttp)),
) {}
```

## Bring an existing server

You don’t need Effect. Point `main` at your own Node server, such as a Hono app. Alchemy bundles it as it is, with no wrapper and no Effect runtime. Listen on the address Lambda passes in `AWS_LAMBDA_HTTP_ENDPOINT`:

```typescript
import { serve } from "@hono/node-server";
import { Hono } from "hono";

const app = new Hono();
app.get("/", (c) => c.text(\`Hello from ${c.req.path}\`));

const [hostname, port] = (process.env.AWS_LAMBDA_HTTP_ENDPOINT ?? "localhost:3000").split(":");
serve({ fetch: app.fetch, hostname, port: Number(port) });
```

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  { providers: AWS.providers(), state: AWS.state() },
  Effect.gen(function* () {
    const api = yield* AWS.Lambda.WebFunction("Api", {
      main: "./src/server.ts",
      authType: "ApplicationManaged",
    });
    return { url: api.url };
  }),
);
```

Express, Fastify and plain `node:http` work the same way:

```typescript
import express from "express";

const app = express();
app.get("/", (_req, res) => res.send("Hello from Express"));

const [host, port] = (process.env.AWS_LAMBDA_HTTP_ENDPOINT ?? "localhost:3000").split(":");
app.listen(Number(port), host);
```

For a framework’s standalone build output, set `bundle: false` to deploy the directory as it is. Lambda starts `main`, so it must listen on `AWS_LAMBDA_HTTP_ENDPOINT` too:

```typescript
const site = yield* AWS.Lambda.WebFunction("Site", {
  main: "./.output/server/index.mjs",
  bundle: false,
  authType: "ApplicationManaged",
});
```

## Tune concurrency and traffic

Set how many requests one environment serves, the per-request timeout, and caps on environments and requests per second:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "ApplicationManaged",
  maxConcurrencyPerEnvironment: 128,
  timeout: Duration.minutes(5),
  maxEnvironments: 10,
  rateLimit: 500,
});
```

`maxConcurrencyPerEnvironment` is a cap per environment, not a total: at most `maxEnvironments × maxConcurrencyPerEnvironment` requests are in flight. Lower it for CPU-heavy servers. Requests beyond `rateLimit` get HTTP 429. A value Lambda doesn’t accept, such as `rateLimit: 1500`, fails at plan with the allowed range.

Each environment has 2 GB of memory and 2 vCPUs, and you can’t change that. Node’s heap defaults to 1 GB; raise it with `env: { NODE_OPTIONS: "--max-old-space-size=1536" }`.

## Roll out gradually

Set `canary` to send a share of traffic to each new deploy while the previous revision keeps the rest:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "ApplicationManaged",
  canary: { weight: 10 },
});
```

The next deploy that changes code or configuration sends 10% of requests to the new revision and 90% to the one before. Changing `weight` shifts traffic without publishing a new revision. Set it to `100`, or remove `canary`, to promote the new revision. Set it to `0` to roll back: all traffic returns to the previous revision.

Deploying again during a canary replaces the canary and keeps the previous revision, so traffic never lands on more than two revisions. `api.revisionWeights` shows the current split. Lambda splits each request on its own, so one client can reach both revisions; keep the two compatible with each other.

Each endpoint has its own split. Set `canary` on a `WebFunctionEndpoint` to roll it out on its own schedule; an endpoint without `canary` moves to each new revision at once.

## Serve from several Regions

List extra Regions to get one global domain that routes each request to the nearest Region:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "ApplicationManaged",
  regions: ["us-east-1", "eu-west-1"],
});
```

Changing `regions` gives the endpoint a new URL.

## Require signed requests

Set `authType: "IamAuth"` to accept only SigV4-signed requests from callers allowed to call `lambda:InvokeWebFunctionEndpoint`, such as a CloudFront distribution:

```typescript
const internal = yield* AWS.Lambda.WebFunction("Internal", {
  main: "./src/server.ts",
  authType: "IamAuth",
});
```

To let a specific caller in, such as a role in another account, grant it access to the endpoint:

```typescript
yield* AWS.Lambda.WebFunctionPermission("PartnerInvoke", {
  endpoint: internal,
  principal: "arn:aws:iam::444455556666:role/partner-caller",
});
```

## Put CloudFront in front

Serve the function through CloudFront for a custom domain, caching, or AWS WAF. Set `webFunction` on the origin and keep `authType: "IamAuth"`:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "IamAuth",
});

const cdn = yield* AWS.CloudFront.Distribution("Cdn", {
  origins: [{ id: "api", webFunction: api }],
  defaultCacheBehavior: { targetOriginId: "api" },
});
```

CloudFront signs every request to the function, and only this distribution may call it, so requests to the function’s own URL are refused. Requests pass through uncached with their method, query string, cookies and headers, including your app’s own `Authorization` header. Set `cachePolicyId` on a behavior to cache a path, such as static assets. This works for a function with `regions` too: Alchemy signs requests to its global domain with SigV4A.

## Reach private resources

Set `vpc` to send the server’s outbound traffic through your VPC, for example to reach a database in private subnets:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "ApplicationManaged",
  vpc: {
    subnetIds: network.privateSubnetIds,
    securityGroupIds: [appSg.groupId],
  },
});
```

Use at least two subnets in different Availability Zones, and allow the security group on the database. Private subnets need a NAT gateway to reach the internet. A function with `vpc` can’t also use `regions`. Remove `vpc` and the next deploy deletes the connector once no endpoint serves through it.

## Add another endpoint

Each endpoint is a separate way in, with its own URL, auth, and limits. Add one with `WebFunctionEndpoint`, for example a public endpoint for webhooks next to a private one:

```typescript
const api = yield* AWS.Lambda.WebFunction("Api", {
  main: "./src/server.ts",
  authType: "IamAuth",
  maxEnvironments: 50,
});

const webhooks = yield* AWS.Lambda.WebFunctionEndpoint("Webhooks", {
  function: api,
  authType: "ApplicationManaged",
  rateLimit: 100,
  maxEnvironments: 2,
});
```

`api.url` is the function’s own endpoint and `webhooks.url` is the new one. Both serve the same code and follow every deploy, but each scales on its own, so a webhook burst can’t use up the other endpoint’s capacity.

For a Web Function declared as a class, like `Api` above, pass the class itself:

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";
import Api from "./src/api.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: AWS.providers(), state: AWS.state() },
  Effect.gen(function* () {
    const api = yield* Api;
    const webhooks = yield* AWS.Lambda.WebFunctionEndpoint("Webhooks", {
      function: Api,
      authType: "ApplicationManaged",
    });
    return { url: api.url, webhooks: webhooks.url };
  }),
);
```

Your server sees which endpoint a request came through in the `lambda-web-endpoint-name` header. Use it to keep each endpoint to its own routes:

```typescript
app.use(async (c, next) => {
  const endpoint = c.req.header("lambda-web-endpoint-name");
  if (endpoint === "Webhooks" && !c.req.path.startsWith("/webhooks/")) {
    return c.notFound();
  }
  await next();
});
```

## Shutdown

Lambda stops idle environments with `SIGTERM` and gives them 1 second. In-flight responses finish first, then init-level finalizers run, so keep them short.
