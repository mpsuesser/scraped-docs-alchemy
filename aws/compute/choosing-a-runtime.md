---
url: https://alchemy.run/aws/compute/choosing-a-runtime
title: "Choosing a runtime"
description: "How Lambda Functions, Lambda Web Functions, ECS, EKS and EC2 differ in how they run your code, scale and cost, and how to choose one for each part of an app."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

AWS gives you several places to run code. They differ in one main way: how much of the machine AWS manages for you, and so how much you decide yourself. Every one of them runs the same kind of Alchemy program, takes the same bindings, and gets its IAM role from them.

Lambdaper invocation

Web FunctionHTTP server

ECSon Fargate

EKSAuto Mode

EC2a machine

your program

you

you

you

you

you

scaling

AWS

AWS

you

you

you

image & runtime

AWS

AWS

you

you

you

orchestration

AWS

AWS

AWS

you

you

OS & patching

AWS

AWS

AWS

AWS

you

hardware

AWS

AWS

AWS

AWS

AWS

From left to right, AWS manages less and you decide more.

## The same program on every runtime

Whichever you pick, you declare it with an id, its configuration and an Effect. Construction binds what it uses, and the handlers it returns do the work. Here it is as an ECS Service:

```typescript
const api = yield* AWS.ECS.Service(
  "Api",
  { cluster, main: import.meta.url, port: 3000, loadBalancer: true },
  Effect.gen(function* () {
    const getObject = yield* AWS.S3.GetObject(Files);
    return {
      fetch: Effect.succeed(HttpServerResponse.text("Hello from ECS!")),
    };
  }).pipe(Effect.provide(AWS.S3.GetObjectHttp)),
);
```

The same Effect, with its bindings and its `fetch`, runs as a Lambda Function or on EC2.

So moving part of an app from one runtime to another changes its declaration, not its code. Choose by how the work behaves, and change your mind later if it behaves differently than you expected.

## Lambda Functions: run per invocation

A [Lambda Function](lambda.md) runs when something invokes it: a request, a queue message, a new object, a schedule. AWS starts environments as invocations arrive and stops them when they don’t, so an idle Function costs nothing and a busy one scales with no setup.

That makes it an easy and cheap place to start. Its limits come from the same model: an invocation runs for at most 15 minutes, each environment handles one invocation at a time, and a new environment takes a cold start before its first one.

Choose it for event handlers, background jobs, scheduled work, and APIs that see uneven traffic.

<svg viewBox="0 0 692.2 102" style="max-width: 796px" role="img" aria-label="One invocation per environment. An invocation that arrives while the others are busy starts a new environment, with a cold start."><g><text fill="currentColor" x="4" y="34">environment 1</text> <g><rect stroke="currentColor" fill="none" x="120.2" y="18" width="44.8" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="165" y="18" width="145.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="237.79999999999998" y="34" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="355.40000000000003" y="18" width="145.5999999999999" height="24" rx="5"></rect><text fill="currentColor" x="428.2" y="34" text-anchor="middle">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="692.2" y2="52"></line><text fill="currentColor" x="4" y="78">environment 2</text> <g><rect stroke="currentColor" fill="none" x="209.8" y="62" width="44.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="254.60000000000002" y="62" width="145.59999999999997" height="24" rx="5"></rect><text fill="currentColor" x="327.4" y="78" text-anchor="middle">invocation</text></g></g></svg> <svg viewBox="0 0 330 122" style="max-width: 379px" role="img" aria-label="One invocation per environment. An invocation that arrives while the others are busy starts a new environment, with a cold start."><g><text fill="currentColor" x="0" y="22">environment 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="25.44" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="25.44" y="32" width="82.67999999999999" height="24" rx="5"></rect><text fill="currentColor" x="66.78" y="48" text-anchor="middle">invocation</text></g> <g><rect stroke="currentColor" fill="none" x="133.56" y="32" width="82.67999999999998" height="24" rx="5"></rect><text fill="currentColor" x="174.89999999999998" y="48" text-anchor="middle">invocation</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">environment 2</text> <g><rect stroke="currentColor" fill="none" x="50.88" y="86" width="25.43999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="76.32" y="86" width="82.68" height="24" rx="5"></rect><text fill="currentColor" x="117.66" y="102" text-anchor="middle">invocation</text></g></g></svg>

invocationcold start

One invocation per environment. An invocation that arrives while the others are busy starts a new environment, with a cold start.

## Lambda Web Functions: HTTP servers on Lambda

A [Lambda Web Function](web-functions.md) runs an HTTP server on Lambda. One environment serves many requests at once, up to 128, so a spike starts fewer environments and requests share warm connections. Responses stream as your handler produces them.

It only receives HTTP, runs Node.js, and has a fixed 2 GB of memory per environment. Choose it for APIs and websites, and pair it with a Lambda Function when the API hands work to a queue.

<svg viewBox="0 0 704.8 118" style="max-width: 811px" role="img" aria-label="The same traffic in one environment. Requests run side by side and share its warm connections."><g><rect stroke="currentColor" fill="none" x="124.80000000000001" y="11" width="576" height="98" rx="8"></rect><text fill="currentColor" x="4" y="58">environment 1</text> <text fill="currentColor" x="4" y="72">up to 128 at once</text> <g><rect stroke="currentColor" fill="none" x="132.8" y="18" width="44.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="177.60000000000002" y="18" width="156.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="256" y="34" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="222.40000000000003" y="48" width="156.8" height="24" rx="5"></rect><text fill="currentColor" x="300.80000000000007" y="64" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="256" y="78" width="179.20000000000005" height="24" rx="5"></rect><text fill="currentColor" x="345.6" y="94" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="356.8" y="18" width="167.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="440.79999999999995" y="34" text-anchor="middle">request</text></g></g></svg> <svg viewBox="0 0 330 128" style="max-width: 379px" role="img" aria-label="The same traffic in one environment. Requests run side by side and share its warm connections."><g><rect stroke="currentColor" fill="none" x="0" y="29" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="22">environment 1 <tspan fill="currentColor" dx="8">up to 128 at once</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="32" width="25.44" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="25.44" y="32" width="89.03999999999999" height="24" rx="5"></rect><text fill="currentColor" x="69.96" y="48" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="50.88" y="62" width="89.04000000000002" height="24" rx="5"></rect><text fill="currentColor" x="95.4" y="78" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="69.96000000000001" y="92" width="101.75999999999999" height="24" rx="5"></rect><text fill="currentColor" x="120.84" y="108" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="127.2" y="32" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="174.9" y="48" text-anchor="middle">request</text></g></g></svg>

The same traffic in one environment. Requests run side by side and share its warm connections.

## ECS: containers that keep running

[ECS](ecs.md) runs your program as a container on Fargate, and keeps it running. There’s no time limit and no cold start per request, and the program can hold connections, WebSockets or in-memory state for as long as it’s up.

You pay for running tasks whether or not they’re busy. They run inside a VPC: your account’s default one to start, or one you build with the `Network` helper. Alchemy builds the image from your program, or from a Dockerfile, and pushes it for you.

Choose it for long-lived servers, workers that hold connections open, and anything that already ships as a container.

<svg viewBox="0 0 655.2 146" style="max-width: 753px" role="img" aria-label="Tasks run whether or not requests arrive, so there's no cold start, and you pay for every hour they run."><g><text fill="currentColor" x="4" y="34">requests</text> <g><rect stroke="currentColor" fill="none" x="139.2" y="18" width="33.60000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="206.40000000000003" y="18" width="33.599999999999966" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="497.59999999999997" y="18" width="33.60000000000008" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="655.2" y2="52"></line><text fill="currentColor" x="4" y="78">task 1</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="62" width="560" height="24" rx="5"></rect><text fill="currentColor" x="363.2" y="78" text-anchor="middle">running · holds connections and memory</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="655.2" y2="96"></line><text fill="currentColor" x="4" y="122">task 2</text> <g><rect stroke="currentColor" fill="none" x="83.2" y="106" width="560" height="24" rx="5"></rect><text fill="currentColor" x="363.2" y="122" text-anchor="middle">running</text></g></g></svg> <svg viewBox="0 0 330 176" style="max-width: 379px" role="img" aria-label="Tasks run whether or not requests arrive, so there's no cold start, and you pay for every hour they run."><g><text fill="currentColor" x="0" y="22">requests</text> <g><rect stroke="currentColor" fill="none" x="31.8" y="32" width="19.080000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="69.96000000000001" y="32" width="19.079999999999984" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="235.32" y="32" width="19.080000000000013" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">task 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="86" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="102" text-anchor="middle">running · holds connections and memory</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">task 2</text> <g><rect stroke="currentColor" fill="none" x="0" y="140" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="156" text-anchor="middle">running</text></g></g></svg>

Tasks run whether or not requests arrive, so there's no cold start, and you pay for every hour they run.

## EKS: Kubernetes

[EKS](eks.md) runs a Kubernetes cluster, with AWS managing the control plane and, in Auto Mode, the nodes. You deploy Deployments, Jobs, Helm charts and raw manifests from the same Stack, with the same bindings.

A cluster takes about ten minutes to create and costs money while it exists. Choose it when you want Kubernetes itself: its tools, its operators, or a team that already runs on it.

EKS clusterAuto Mode

node 1managed by AWS

webPod

workerPod

node 2managed by AWS

webPod

You declare Pods through Deployments and Jobs. Kubernetes places them on nodes, and Auto Mode adds and patches the nodes.

## EC2: the whole machine

[EC2](ec2.md) gives you a virtual machine. You pick the instance type, the operating system and the disk, and you own patching and recovery. Alchemy can run your program on it directly, or leave it as a plain machine.

Choose it for GPUs, custom daemons, software that expects a host, or steady load where a reserved machine is cheapest.

VPCyour network

public subnetus-east-1a

security groupallows:3000 in

ServerEC2 instance

type

g5.xlarge · 1 GPU

image

your OS and disk

you own

patching, restarts, recovery

The whole machine is yours: its size, its operating system, everything on it, and keeping it healthy.

## Mix runtimes in one app

An app rarely needs just one. A common shape is a Web Function for the API, a Lambda Function consuming the queue it writes to, and an ECS service for the one process that must stay connected:

<svg viewBox="0 0 652 204" style="max-width: 750px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-c464t3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 172 102 C 209 102, 209 56, 246 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-c464t3)"></path></g><g><path d="M 400 56 C 437 56, 437 102, 474 102" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-c464t3)"></path></g><g><path d="M 172 102 C 209 102, 209 148, 246 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-c464t3)"></path></g><g transform="translate(24, 70)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Web Function</text></g> <g transform="translate(252, 24)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Jobs</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">SQS Queue</text></g> <g transform="translate(480, 70)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Function</text></g> <g transform="translate(252, 116)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Feed</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">ECS Service</text></g></svg> <svg viewBox="0 0 328 332" style="max-width: 377px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-1znibi" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 74 C 164 101, 80 101, 80 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-1znibi)"></path></g><g><path d="M 80 198 C 80 225, 164 225, 164 252" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-1znibi)"></path></g><g><path d="M 164 74 C 164 101, 248 101, 248 128" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-1znibi)"></path></g><g transform="translate(94, 10)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Web Function</text></g> <g transform="translate(10, 134)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Jobs</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">SQS Queue</text></g> <g transform="translate(94, 258)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Lambda Function</text></g> <g transform="translate(178, 134)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Feed</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">ECS Service</text></g></svg>

Each runtime has its own role, written by its own bindings, so each part of the app can reach only what it uses.
