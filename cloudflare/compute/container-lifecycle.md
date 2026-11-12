---
url: https://alchemy.run/cloudflare/compute/container-lifecycle
title: "Container lifecycle"
description: "Start, stop, wait for, and restore a Cloudflare Container from its Durable Object — and save its filesystem as a snapshot to restore later."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A [Container](containers.md) runs only when you start it. You start it, stop it, and restore it from the Durable Object that owns it.

The examples use this container:

```typescript
export class Sandbox extends Cloudflare.Container<Sandbox>()("Sandbox", {
  schedulingPolicy: "durable_object",
  image: "node:24-slim",
}) {}
```

<svg viewBox="-4 28 716.6 200" style="max-width: 824px" role="img" aria-label="Only the Durable Object moves its container between states. start() is safe to call before every operation: it does nothing when the container is already running, and starts it again after it stopped or exited."><defs><marker id="alc-sm-arrow-f4lzzt" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 180.2 198.4139917208752 Q 279.2 206.05999999999997 353.50515816799737 89.3739854279505" marker-end="url(#alc-sm-arrow-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 300.19999999999993 57.58600827912479 Q 201.2 49.940000000000026 126.89484183200258 166.62601457204948" marker-end="url(#alc-sm-arrow-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 384.9463022508038 86 Q 461.4 206.06 556.4118765841854 198.7220040911327" marker-end="url(#alc-sm-arrow-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 615.8536977491962 170 Q 539.4 49.93999999999998 444.38812341581456 57.2779959088673" marker-end="url(#alc-sm-arrow-f4lzzt)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="193" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="193" x2="36" y2="193" marker-end="url(#alc-sm-arrow-f4lzzt)"></line></g><rect stroke="currentColor" fill="none" x="40" y="170" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="197" text-anchor="middle">stopped</text></g> <g><rect stroke="currentColor" fill="none" x="300.19999999999993" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="370.29999999999995" y="61" text-anchor="middle">running</text> <text fill="currentColor" x="370.29999999999995" y="76" text-anchor="middle">serving the DO</text></g> <g><rect stroke="currentColor" fill="none" x="560.4" y="170" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="630.5" y="191" text-anchor="middle">exited</text> <text fill="currentColor" x="630.5" y="206" text-anchor="middle">crashed or killed</text></g> <g><rect stroke="currentColor" fill="none" x="244.9262895419993" y="164.97699428720642" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="273.0262895419993" y="177.97699428720642" text-anchor="middle">start()</text></g> <g><rect stroke="currentColor" fill="none" x="146.27371045800064" y="71.02300571279358" width="122.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="207.37371045800063" y="84.02300571279358" text-anchor="middle">idle or destroy()</text></g> <g><rect stroke="currentColor" fill="none" x="418.1395447087473" y="164.21050102278318" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="466.03954470874726" y="177.21050102278318" text-anchor="middle">process exits</text></g> <g><rect stroke="currentColor" fill="none" x="490.16045529125256" y="71.78949897721682" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="534.7604552912526" y="84.78949897721682" text-anchor="middle">next start()</text></g></svg> <svg viewBox="15.243509361271094 -4 357.1564906387289 378" style="max-width: 411px" role="img" aria-label="Only the Durable Object moves its container between states. start() is safe to call before every operation: it does nothing when the container is already running, and starts it again after it stopped or exited."><defs><marker id="alc-sm-arrow-n-f4lzzt" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 286.770777383341 86 Q 266.44 218.49599999999998 184.17518517157453 209.28974951875315" marker-end="url(#alc-sm-arrow-n-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 113.62922261665894 178 Q 133.95999999999998 45.50400000000002 216.22481482842542 54.71025048124685" marker-end="url(#alc-sm-arrow-n-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 180.2 193.1551132147883 Q 266.44 183.50400000000002 286.16410015298555 312.04627482768865" marker-end="url(#alc-sm-arrow-n-f4lzzt)"></path><path stroke="currentColor" fill="none" d="M 220.19999999999996 346.8448867852117 Q 133.95999999999998 356.496 114.23589984701437 227.95372517231132" marker-end="url(#alc-sm-arrow-n-f4lzzt)"></path><g><g><circle stroke="currentColor" fill="none" cx="290.29999999999995" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="290.29999999999995" y1="20" x2="290.29999999999995" y2="36" marker-end="url(#alc-sm-arrow-n-f4lzzt)"></line></g><rect stroke="currentColor" fill="none" x="220.19999999999996" y="40" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="290.29999999999995" y="67" text-anchor="middle">stopped</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="110.1" y="199" text-anchor="middle">running</text> <text fill="currentColor" x="110.1" y="214" text-anchor="middle">serving the DO</text></g> <g><rect stroke="currentColor" fill="none" x="220.19999999999996" y="316" width="140.2" height="46" rx="23"></rect><text fill="currentColor" x="290.29999999999995" y="337" text-anchor="middle">exited</text> <text fill="currentColor" x="290.29999999999995" y="352" text-anchor="middle">crashed or killed</text></g> <g><rect stroke="currentColor" fill="none" x="250.95649063872887" y="173.0704373796883" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="255.95649063872887" y="186.0704373796883" text-anchor="start">start()</text></g> <g><rect stroke="currentColor" fill="none" x="27.243509361271094" y="70.92956262031173" width="122.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="144.44350936127108" y="83.92956262031173" text-anchor="end">idle or destroy()</text></g> <g><rect stroke="currentColor" fill="none" x="249.81102503824638" y="208.05234701061926" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="254.81102503824638" y="221.05234701061926" text-anchor="start">process exits</text></g> <g><rect stroke="currentColor" fill="none" x="61.388974961753576" y="311.94765298938074" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="145.58897496175356" y="324.94765298938074" text-anchor="end">next start()</text></g></svg>

Only the Durable Object moves its container between states. start() is safe to call before every operation: it does nothing when the container is already running, and starts it again after it stopped or exited.

## Bind the container

`yield* Sandbox` in a Durable Object’s Construction phase gives you the container’s handle:

```typescript
export default class Workspace extends Cloudflare.DurableObject<Workspace>()(
  "Workspace",
  Effect.gen(function* () {
    const sandbox = yield* Sandbox;

    return Effect.gen(function* () {
      return {};
    });
  }),
) {}
```

Binding does not start the container.

## Start it before you use it

Call `start` at the top of each operation that needs the container:

```typescript
const start = sandbox.start({
  entrypoint: ["sleep", "infinity"],
  enableInternet: false,
});

return {
  run: (cmd: string[]) =>
    Effect.gen(function* () {
      yield* start;
      // the container is running
    }),
};
```

`start` does nothing when the container is already running, and concurrent calls start it once. A container that stopped or crashed since the last request starts again on the next one.

The options only apply when `start` actually starts the container. To run with different options, stop the container first.

## Choose a size per start

A container with `schedulingPolicy: "durable_object"` picks its size when it starts:

```typescript
yield* sandbox.start({
  instance: "standard-1",
  entrypoint: ["sleep", "infinity"],
  enableInternet: false,
});
```

`start` starts the Container’s own image. To run another image in the same Durable Object, bind another Container — see [Several images in one Durable Object](containers.md#several-images-in-one-durable-object). An async Worker attaches Containers to a class with `Cloudflare.DurableObject(className, { containers })` — see [Several images on an async Worker](containers.md#several-images-on-an-async-worker).

## Start it from a Layer

A Layer that depends on the container starts it the same way:

```typescript
export const ShellLive = Layer.effect(
  Shell,
  Effect.gen(function* () {
    const sandbox = yield* Sandbox;
    const start = sandbox.start({ enableInternet: false });

    return {
      run: (cmd: string[]) =>
        start.pipe(
          Effect.andThen(sandbox.exec(cmd)),
          Effect.flatMap((child) => child.output()),
          Effect.scoped,
        ),
    };
  }),
);
```

Every Layer in the Durable Object shares one container, so a Layer never stops it.

## Wait for a port

`getTcpPort` waits until the server inside the container accepts connections:

```typescript
const { fetch } = yield* sandbox.getTcpPort(8080);

const health = Effect.gen(function* () {
  yield* start;
  const response = yield* fetch(HttpClientRequest.get("http://container/health"));
  return yield* response.text;
});
```

A port never starts the container. If the container is stopped, the request fails with `ContainerNotRunningError`.

## Run a command

`exec` runs a command in a started container:

```typescript
const output = yield* Effect.scoped(
  Effect.gen(function* () {
    const child = yield* sandbox.exec(["npm", "test"]);
    return yield* child.output();
  }),
);
```

`exec` runs the command without a shell. Pass `["sh", "-c", command]` when you need one. Closing the scope kills a process that is still running.

## Check whether it is running

```typescript
if (yield* sandbox.running) {
  // the process is up; its ports may not be ready yet
}
```

## Stop it

`destroy` stops the container immediately:

```typescript
yield* sandbox.destroy();
```

`signal` sends a signal to its main process instead, so it can shut down on its own:

```typescript
yield* sandbox.signal(15); // SIGTERM
```

## Let it sleep

When nothing calls the Durable Object, it goes idle and its container stops. Change how long it waits with `setInactivityTimeout`:

```typescript
yield* sandbox.setInactivityTimeout(10 * 60 * 1000); // 10 minutes
```

The next operation that calls `start` starts the container again.

## Watch for a crash

`monitor` waits until the container exits:

```typescript
yield* sandbox.monitor().pipe(
  Effect.catchTag("ContainerError", (error) =>
    Effect.logError(\`container exited: ${error.message}\`),
  ),
);
```

A crash fails with `ContainerError`; a clean exit succeeds.

## Save a snapshot

`snapshotContainer` saves the container’s filesystem:

```typescript
const snapshot = yield* sandbox.snapshotContainer({ name: "workspace" });
yield* state.storage.put("snapshot", snapshot);
```

Store the snapshot in the Durable Object’s storage so you can restore it after the container stops. A snapshot holds the writable root filesystem: files, installed packages, build caches. It does not hold memory, running processes, or mounted filesystems.

Snapshots need `schedulingPolicy: "durable_object"`.

## Restore a snapshot

Pass the snapshot to `start` to begin from that filesystem:

```typescript
const snapshot = yield* state.storage.get<Cloudflare.Containers.ContainerSnapshot>("snapshot");

yield* sandbox.start({
  ...(snapshot ? { containerSnapshot: snapshot } : {}),
  entrypoint: ["sleep", "infinity"],
  enableInternet: false,
});
```

Without a snapshot, `start` uses the Container’s image. The entrypoint starts fresh. A snapshot can start any number of containers, and expires after 30 days without a restore.

## Snapshot before you stop

Save the filesystem, then stop the container, so the next start picks up where this one left off:

```typescript
const suspend = Effect.gen(function* () {
  const snapshot = yield* sandbox.snapshotContainer({ name: "workspace" });
  yield* state.storage.put("snapshot", snapshot);
  yield* sandbox.destroy();
});
```

## Where next

- [Containers](containers.md) — declare a container and its image.
- [Run a Container](run-a-container.md) — build a shell-running container end to end.
- [Durable Objects](durable-objects.md) — the object that owns each container.
- [Container API reference](https://alchemy.run/providers/cloudflare/containers#container)
