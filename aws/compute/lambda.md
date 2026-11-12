---
url: https://alchemy.run/aws/compute/lambda
title: "Lambda"
description: "How a Lambda Function runs your code per invocation, how it's invoked over a Function URL, by events or by other code, and how construction, timeouts and configuration work."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Lambda Function is code that AWS runs when something invokes it: an HTTP request, a message on a queue, a new object in a Bucket, a schedule. You don’t run a server. AWS starts environments for your code as invocations arrive, and you pay only for the time they spend running.

## How Lambda runs your code

Lambda runs your code in **execution environments**: small, isolated sandboxes, each holding one copy of your Function. An environment handles one invocation at a time. When invocations arrive faster than the existing environments can finish them, Lambda starts more, and when traffic drops, it stops them.

<svg viewBox="0 0 692.2 168" style="max-width: 796px" role="img" aria-label="Each environment runs one invocation at a time. When more arrive than the warm ones can take, Lambda starts more, each with a cold start first."><line stroke-opacity="0.2" stroke="currentColor" x1="198.6" y1="24" x2="198.6" y2="162"></line><g><text fill="currentColor" x="4" y="56">environment 1</text> <g><rect stroke="currentColor" fill="none" x="120.2" y="40" width="39.2" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="159.4" y="40" width="106.4" height="24" rx="5"></rect><text fill="currentColor" x="212.60000000000002" y="56" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="277" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="327.4" y="56" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="400.2" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="450.59999999999997" y="56" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="534.6" y="40" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="585" y="56" text-anchor="middle">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="692.2" y2="74"></line><text fill="currentColor" x="4" y="100">environment 2</text> <g><rect stroke="currentColor" fill="none" x="198.6" y="84" width="39.20000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="237.8" y="84" width="106.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="291" y="100" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="366.6" y="84" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="417" y="100" text-anchor="middle">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="692.2" y2="118"></line><text fill="currentColor" x="4" y="144">environment 3</text> <g><rect stroke="currentColor" fill="none" x="209.8" y="128" width="39.19999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="249" y="128" width="106.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="302.20000000000005" y="144" text-anchor="middle">invocation</text></g></g> <text fill="currentColor" x="198.6" y="18" text-anchor="middle">traffic spikes</text></svg> <svg viewBox="0 0 330 228" style="max-width: 379px" role="img" aria-label="Each environment runs one invocation at a time. When more arrive than the warm ones can take, Lambda starts more, each with a cold start first."><line stroke-opacity="0.2" stroke="currentColor" x1="44.519999999999996" y1="24" x2="44.519999999999996" y2="222"></line><g><text fill="currentColor" x="0" y="44">environment 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="22.259999999999998" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="22.259999999999998" y="54" width="60.42000000000001" height="24" rx="5"></rect><text fill="currentColor" x="22.259999999999998" y="90" text-anchor="start">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="89.03999999999999" y="54" width="57.24000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="159" y="54" width="57.23999999999998" height="24" rx="5"></rect><text fill="currentColor" x="159" y="90" text-anchor="start">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="235.32" y="54" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="235.32" y="90" text-anchor="start">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">environment 2</text> <g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="123" width="22.260000000000005" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="66.78" y="123" width="60.42" height="24" rx="5"></rect><text fill="currentColor" x="66.78" y="159" text-anchor="start">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="139.92000000000002" y="123" width="57.23999999999998" height="24" rx="5"></rect><text fill="currentColor" x="203.16" y="139" text-anchor="start">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="168" x2="330" y2="168"></line><text fill="currentColor" x="0" y="182">environment 3</text> <g><rect stroke="currentColor" fill="none" x="50.88" y="192" width="22.259999999999998" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="73.14" y="192" width="60.42" height="24" rx="5"></rect><text fill="currentColor" x="139.56" y="208" text-anchor="start">invocation</text></g></g> <text fill="currentColor" x="44.519999999999996" y="18" text-anchor="middle">traffic spikes</text></svg>

invocationcold start

Each environment runs one invocation at a time. When more arrive than the warm ones can take, Lambda starts more, each with a cold start first.

Starting a new environment is a **cold start**: Lambda loads your code and runs its setup before the first invocation, which adds tens to hundreds of milliseconds. After that the environment is **warm**, and later invocations skip the setup.

So a Function scales with no capacity planning, and costs nothing while nothing calls it. In exchange, an invocation can run for at most 15 minutes, and memory, and the CPU that comes with it, is fixed per Function.

## Declare a Function

A Function is a class. Its first argument is the id, its second the configuration, and its third the program it runs:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as Effect from "effect/Effect";

export default class Api extends AWS.Lambda.Function<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    return {};
  }),
) {}
```

`main: import.meta.url` makes this file the entry point. Deploy bundles it into a zip, uploads it, and creates the Function with its execution role and log group. The role starts with permission to write logs, and each [binding](../iam.md) adds to it.

Yield the class in your Stack to deploy it:

```typescript
const api = yield* Api;
```

## Construction runs once per environment

The program has two parts. The outer `Effect.gen` is the **Construction phase**. It runs once when an environment starts, and again at deploy time, where its bindings become IAM statements. What it returns are the handlers, which run once per invocation:

```typescript
Effect.gen(function* () {
  const putObject = yield* AWS.S3.PutObject(Files); // once per environment
  return {
    fetch: putObject({ Key: "hit", Body: "1" }).pipe(/* once per invocation */),
  };
});
```

Put clients and bindings in Construction, so every invocation in the environment reuses them. Put per-request work in the handlers.

## Serve HTTP from a Function URL

A `fetch` handler answers HTTP requests. It’s an Effect that reads the request and returns a response, with the `effect/http` APIs:

```typescript
return {
  fetch: Effect.succeed(HttpServerResponse.text("Hello from Lambda!")),
};
```

Requests reach it through a **Function URL**, an HTTPS endpoint AWS gives the Function. Choose who may call it with `functionUrl`:

```typescript
{ main: import.meta.url, functionUrl: true }
```

`true` creates a public URL, `{ authType: "AWS_IAM" }` creates one that only accepts requests signed with AWS credentials, and `false` creates none. Leaving it out creates a public URL, so set `functionUrl: false` on a Function that only handles events.

Return the URL from your Stack to see it after a deploy:

```typescript
const api = yield* Api;
return { url: api.functionUrl };
```

A Lambda Function buffers its whole response, up to 6 MB, before the caller sees any of it. For an API or a website that streams, or that serves many requests at once per environment, use a [Lambda Web Function](web-functions.md).

## Be invoked by events

Services invoke a Function too. A Queue delivers batches of messages, a Bucket reports new objects, a Table streams its changes, and EventBridge delivers events and schedules. Subscribe to one from Construction:

```typescript
yield* SQS.consumeQueueMessages(Jobs, { batchSize: 10 }, (event) =>
  Effect.forEach(event.Records, (record) => Console.log(record.body), { discard: true }),
);
```

Deploy creates the subscription and the permissions it needs, and provides the event source’s layer, here `AWS.Lambda.QueueEventSource`, to the Function. See [SQS](../messaging/sqs.md), [S3 events](../messaging/s3-events.md), [DynamoDB Streams](../messaging/dynamodb-streams.md), [Kinesis](../messaging/kinesis.md) and [EventBridge](../messaging/eventbridge.md).

## Be invoked by other code

Another runtime can call a Function directly. Bind `Lambda.InvokeFunction` to it, and the caller’s role gets `lambda:InvokeFunction` on that Function only:

```typescript
const invoke = yield* AWS.Lambda.InvokeFunction(Worker);
yield* invoke({ InvocationType: "Event", Payload: JSON.stringify({ id }) });
```

`"Event"` hands the payload over and returns at once. `"RequestResponse"` waits for the result. For typed calls between Functions, see [Effect RPC](../apis/effect-rpc.md).

## Configure per stage

Memory, timeout and log retention are props. Memory also sets the CPU: a Function with 1,769 MB gets one full vCPU.

```typescript
{
  main: import.meta.url,
  memorySize: 1024,
  timeout: Duration.seconds(30),
  logging: { retention: "2 weeks" },
}
```

To vary them by stage, compute the props from the Stack:

```typescript
Stack.useSync((stack) => ({
  main: import.meta.url,
  memorySize: stack.stage === "prod" ? 1024 : 512,
}))
```

Without `logging`, Lambda creates the log group on the first invocation and keeps its events forever. With it, Alchemy creates the group up front with that retention, and deletes it with the Function.

## Clean up inside the invocation

Each invocation runs in a fresh `Scope`, closed before the response is sent. A finalizer you add in a handler runs then:

```typescript
fetch: Effect.gen(function* () {
  yield* Effect.addFinalizer(() => Console.log("request done"));
  return HttpServerResponse.text("ok");
}),
```

There’s no reliable way to do work after the response on Lambda. AWS freezes the environment once the invocation returns, and a frozen environment may never run again. So keep finalizers short, and write anything that must not be lost, to a queue or a table, before you return.

Finalizers added in Construction run when the environment shuts down, with about half a second to finish, and not at all if the environment crashes. Treat them as best effort. See [Instance scope vs request scope](../../infrastructure-as-effects/runtime.md#instance-scope-vs-request-scope).

## When an invocation times out

When a handler runs past its `timeout`, it’s interrupted. Its finalizers run and its open spans end, so the trace shows the call that hung, ending with an `AWS.Lambda.InvocationTimeoutError`.

Lambda itself is given `timeout` plus `timeoutMargin`, one second by default, so cleanup and telemetry finish before Lambda stops the environment. Size the margin for your finalizers and one export to your telemetry backend:

```typescript
{
  main: import.meta.url,
  timeout: Duration.seconds(30),
  timeoutMargin: Duration.seconds(2),
}
```

The two together must fit within Lambda’s 15 minutes. Lambda still records the invocation as timed out, retries it like any other failure, and starts a fresh environment, so nothing the interrupted handler left running can resume later.

## When to choose another runtime

Lambda is a good place to start: there’s nothing to run while idle, and it scales without setup. It fits work that finishes within 15 minutes and arrives as requests or events.

Reach for something else when the work doesn’t fit that shape. An API that streams responses or holds many requests per environment fits a [Lambda Web Function](web-functions.md). A process that runs continuously, holds connections open or already ships as a container fits [ECS](ecs.md). Kubernetes workloads fit [EKS](eks.md), and software that needs the whole machine fits [EC2](ec2.md). See [Choosing a runtime](choosing-a-runtime.md).
