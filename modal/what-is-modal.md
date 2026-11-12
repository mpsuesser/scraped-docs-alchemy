---
url: https://alchemy.run/modal/what-is-modal
title: "What is Modal?"
description: "The concepts behind Modal — workspaces and tokens, environments, apps, images, Functions, Servers, Sandboxes, Volumes, Dicts, Queues and Secrets — and how @alchemy.run/modal models each of them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Modal runs your code in containers, with GPUs when you ask for them. A **workspace** holds everything you create and pays for it. It’s divided into **environments**, and in each environment, **apps** group the code that runs: **Functions** and **Servers** that Modal scales with traffic, and **Sandboxes** your code creates while it runs. Each of them starts from an **image**. Volumes, Dicts, Queues and Secrets sit beside the apps in the same environment. You pay per second for the containers that run, and nothing while they’re scaled to zero.

Take one small app: a Cloudflare Worker gives each user a Sandbox from the `Warm` image, with a Volume of their own as `/home/user`. An `Embed` Function on an L4 GPU turns their text into vectors. The `Coder` and `Inference` apps hold the code, and the `MyEnv` environment holds both apps with their Volume and Secret.

Apiyour Cloudflare Worker

acmeworkspace

MyEnvenvironment

Coderapp

Warmimage

session-samSandbox

session-anaSandbox

home-samVolume

home-anaVolume

HuggingFaceSecret

Inferenceapp

EmbedFunction

gpu

L4

Your Worker creates a Sandbox and a Volume for each person using your app, here sam and ana, and there can be as many as you need. Every Sandbox boots from the one Warm image and mounts its own Volume. The apps, the image, the Embed Function and its HuggingFace Secret are deployed once, in one environment of the workspace.

## Workspaces: who owns and pays

A **workspace** owns your apps, storage and secrets, its members, and the bill. A personal account is a workspace of one, and a team shares one.

Modal authenticates calls with a **token**: a token id (`ak-…`) and a secret (`as-…`). A token acts on the whole workspace, so Alchemy deploys with yours, and gives each runtime that binds a Modal object a **service user** of its own, with only the role its bindings need.

```sh
pnpm alchemy profile edit --add Modal
```

See [Workspaces & tokens](workspaces-and-tokens.md).

An **environment** divides a workspace. Two objects with the same name in two environments are different objects, so `dev` and `prod` never share a Volume by accident. Every workspace starts with one called `main`.

```typescript
export const MyEnv = Modal.Environment("MyEnv", {});
```

Alchemy names the environment from your app, stage and id, so each stage gets its own. Everything else you declare names its environment with `environment: MyEnv`, or lands in the workspace’s default one.

See [Workspaces & tokens](workspaces-and-tokens.md#environments-keep-stages-apart).

## Apps: code deployed and stopped together

An **app** groups the code you deploy together and acts as its namespace. A deployed app runs until you stop it, and stopping it stops every Function, Server and Sandbox in it.

```typescript
export const Coder = Modal.App("Coder", { environment: MyEnv });
```

## Images: the filesystem a container starts from

An **image** is the filesystem a container starts from. Modal builds it on its own builders, from a registry image, a Dockerfile, or another image with commands run on top, so a deploy needs no Docker on your machine.

```typescript
export const Node = Modal.Image("Node", { app: Coder, image: "node:22-bookworm" });

export const Warm = Modal.Image("Warm", {
  app: Coder,
  from: Node,
  run: ["git clone --depth 1 https://github.com/sindresorhus/is /work"],
});
```

`Warm` runs its setup once at deploy and saves the result, so every container from it starts with the repository already there.

See [Images](sandboxes/images.md).

## Functions: containers that scale with calls

A **Function** is code Modal runs when it’s called. Each call is an **input**. Modal starts containers as inputs arrive, runs several at once when calls pile up, and stops them once they’ve been idle for the scale-down window. With no inputs, nothing runs and nothing is billed.

<svg viewBox="0 0 677.4 124" style="max-width: 779px" role="img" aria-label="The first call starts a container. When calls arrive faster than one container serves them, Modal starts another. Each stops after it has been idle for the scale-down window, and with no calls nothing runs."><line stroke-opacity="0.2" stroke="currentColor" x1="273.4" y1="24" x2="273.4" y2="118"></line><g><text fill="currentColor" x="4" y="56">container 1</text> <g><rect stroke="currentColor" fill="none" x="127.80000000000001" y="40" width="56" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="183.8" y="40" width="100.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="234.20000000000002" y="56" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="295.79999999999995" y="40" width="100.80000000000007" height="24" rx="5"></rect><text fill="currentColor" x="346.2" y="56" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="396.6" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="452.6" y="56" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="100">container 2</text> <g><rect stroke="currentColor" fill="none" x="273.4" y="84" width="56" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="329.4" y="84" width="100.79999999999995" height="24" rx="5"></rect><text fill="currentColor" x="379.79999999999995" y="100" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="430.19999999999993" y="84" width="112.00000000000011" height="24" rx="5"></rect><text fill="currentColor" x="486.2" y="100" text-anchor="middle">idle</text></g></g> <text fill="currentColor" x="273.4" y="18" text-anchor="middle">calls pile up</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="The first call starts a container. When calls arrive faster than one container serves them, Modal starts another. Each stops after it has been idle for the scale-down window, and with no calls nothing runs."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="138"></line><g><text fill="currentColor" x="0" y="44">container 1</text> <g><rect stroke="currentColor" fill="none" x="12.72" y="54" width="31.799999999999997" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="54" width="57.24000000000001" height="24" rx="5"></rect><text fill="currentColor" x="73.14" y="70" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="108.11999999999999" y="54" width="57.24000000000002" height="24" rx="5"></rect><text fill="currentColor" x="136.74" y="70" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="165.36" y="54" width="63.599999999999966" height="24" rx="5"></rect><text fill="currentColor" x="197.16" y="70" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">container 2</text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="108" width="31.80000000000001" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="127.2" y="108" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="155.82" y="124" text-anchor="middle">embed</text></g> <g><rect stroke="currentColor" fill="none" x="184.44" y="108" width="63.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="216.24" y="124" text-anchor="middle">idle</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">calls pile up</text></svg>

callcold startidle

The first call starts a container. When calls arrive faster than one container serves them, Modal starts another. Each stops after it has been idle for the scale-down window, and with no calls nothing runs.

In Alchemy, a Function is a class that holds its own code. Every function it returns is a method callers reach by name:

```typescript
export default class Embed extends Modal.Function<Embed>()(
  "Embed",
  { main: import.meta.url, app: Inference, gpu: "L4" },
  Effect.gen(function* () {
    return {
      embed: Effect.fn(function* (text: string) {
        return { tokens: text.split(/\s+/).length };
      }),
    };
  }),
) {}
```

A Function can also run on a schedule, or serve its `fetch` handler at a `*.modal.run` URL as a **web endpoint**.

See [Functions](services/functions.md) and [Web endpoints](services/web-endpoints.md).

## Servers: HTTP that scales with concurrency

A **Server** is a long-running HTTP server. Requests reach it through one routing region, and Modal scales its containers on how many requests each is serving at once, not on a queue of calls.

```typescript
export default class Api extends Modal.Server<Api>()(
  "Api",
  { main: import.meta.url, app: Backend, routingRegion: "us-east" },
  Effect.gen(function* () {
    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }),
) {}
```

See [Servers](services/servers.md) and [Choosing a runtime](compute/choosing-a-runtime.md).

## Sandboxes: containers your code creates on the fly

A **Sandbox** is a container your code creates while it runs, to execute code it doesn’t trust: an agent’s commands, a user’s script. It runs until its timeout (five minutes unless you ask for up to 24 hours) or until it’s been idle for its idle timeout, and you run commands in it, read and write its files, and reach its ports through tunnels.

```typescript
const sandboxes = yield* Modal.CreateSandbox(Warm);
const sandbox = yield* sandboxes.create({ timeout: "10 minutes" });
const result = yield* sandbox.exec(["ls", "/work"]);
```

A Sandbox’s filesystem, or its memory too, can be snapshotted and started again elsewhere.

See [Sandboxes](sandboxes/sandboxes.md) and [Snapshots](sandboxes/snapshots.md).

## Volumes, Dicts and Queues: state between containers

A **Volume** is a filesystem that Functions, Servers and Sandboxes mount at a path, and that outlives every container that mounts it. A **Dict** is a key-value store and a **Queue** hands work between processes; both hold JSON and expire their entries.

```typescript
export const Weights = Modal.Volume("Weights", { environment: MyEnv });
export const Results = Modal.Dict("Results", { environment: MyEnv });
export const Jobs = Modal.Queue("Jobs", { environment: MyEnv });
```

A Volume per user is created by name while your app runs:

```typescript
const volumes = yield* Modal.CreateVolume(MyEnv);
const home = yield* volumes.getOrCreate(\`home-${user}\`);
```

See [Volumes](storage/volumes.md), [Dicts](storage/dicts.md) and [Queues](storage/queues.md).

## Secrets: variables a container gets

A **Secret** is a named set of environment variables, stored in an environment and attached to the containers that need them. A Sandbox can also keep a Secret out of its environment entirely: Modal adds it to outgoing requests for the hosts you name, so code inside never sees it.

```typescript
export const HuggingFace = Modal.Secret("HuggingFace", {
  environment: MyEnv,
  env: { HF_TOKEN: Config.Redacted("HF_TOKEN") },
});
```

See [Secrets](security/secrets.md).

## Network: what a Sandbox can reach

A Sandbox reaches the internet unless you limit it. Block its network, or allow only some domains and address ranges, and change either while it runs. Its ports reach the outside through **tunnels**, at a public URL per port.

```typescript
const sandbox = yield* sandboxes.create({ allowedDomains: ["github.com"], encryptedPorts: [3000] });
```

See [Network & tunnels](networking/network.md).

## State: where Alchemy keeps its record

Modal has no storage for Alchemy’s own record of what each Stack created. Keep it on disk with `Alchemy.localState()`, or share it with your team and CI through `Cloudflare.state()` or `AWS.state()`.

## How @alchemy.run/modal models these concepts

Your profile holds a workspace token, and `Modal.providers()` deploys with it. Environments, apps, images, Volumes, Dicts, Queues and Secrets an app shares are resources in the Stack, and Functions, Servers and Sandbox programs are classes that hold their own code. Sandboxes, Volumes, Secrets and images made per user or task come from bindings your runtime uses while it runs (`CreateSandbox`, `CreateVolume`, `CreateSecret`, `CreateImage`). They carry their stack, stage and binding in tags or a name prefix, and stay when the Stack is destroyed. Each runtime that binds a Modal object gets a service user of its own, never your token.

Next, build one: the [tutorial](tutorial/part-1.md) puts these ideas to work, one step at a time.
