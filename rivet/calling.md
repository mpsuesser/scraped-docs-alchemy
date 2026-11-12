---
url: https://alchemy.run/rivet/calling
title: "Calling a Durable Object"
description: "How any runtime calls a Rivet Durable Object by name — methods, streams, typed errors, HTTP requests, and calls from one Durable Object to another."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A caller reaches a room by naming it. It doesn’t need to know where the room runs, or whether it’s running at all: it sends the call to the control plane, and the control plane finds the room or starts it.

<svg viewBox="0 0 500 285" style="max-width: 575px" role="img" aria-label="The caller names the room and the gateway does the rest: it starts the room if it's asleep, then passes the call on. The caller never needs a route to the nodes."><defs><marker id="alc-seq-arrow-zzqx6p" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-zzqx6p" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="281"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">Worker or Lambda</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="281"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Gateway</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">control plane</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="281"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">lobby</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">Room, on a node</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="59.7" y="66" width="212.6" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">join() on lobby, with the token</text></g> <g><rect stroke="currentColor" x="188.5" y="113" width="123" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="129" text-anchor="middle">lobby is asleep</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="161" x2="412" y2="161" marker-end="url(#alc-seq-arrow-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="264" y="142" width="140" height="15" rx="3"></rect><text fill="currentColor" x="334" y="154" text-anchor="middle">start on a free node</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="201" x2="412" y2="201" marker-end="url(#alc-seq-arrow-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="310.2" y="182" width="47.599999999999994" height="15" rx="3"></rect><text fill="currentColor" x="334" y="194" text-anchor="middle">join()</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="241" x2="88" y2="241" marker-end="url(#alc-seq-arrow-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="242.7" y="222" width="14.6" height="15" rx="3"></rect><text fill="currentColor" x="250" y="234" text-anchor="middle">4</text></g></svg> <svg viewBox="0 0 322 284" style="max-width: 370px" role="img" aria-label="The caller names the room and the gateway does the rest: it starts the room if it's asleep, then passes the call on. The caller never needs a route to the nodes."><defs><marker id="alc-seq-arrow-n-zzqx6p" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-zzqx6p" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="37" y1="58" x2="37" y2="280"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="70" height="56" rx="8"></rect><text fill="currentColor" x="37" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="37" y="37" text-anchor="middle">Worker or</text> <text fill="currentColor" x="37" y="50" text-anchor="middle">Lambda</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="58" x2="161" y2="280"></line><rect stroke="currentColor" fill="none" x="126" y="2" width="70" height="56" rx="8"></rect><text fill="currentColor" x="161" y="22" text-anchor="middle">Gateway</text> <text fill="currentColor" x="161" y="37" text-anchor="middle">control</text> <text fill="currentColor" x="161" y="50" text-anchor="middle">plane</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="285" y1="58" x2="285" y2="280"></line><rect stroke="currentColor" fill="none" x="250" y="2" width="70" height="56" rx="8"></rect><text fill="currentColor" x="285" y="22" text-anchor="middle">lobby</text> <text fill="currentColor" x="285" y="37" text-anchor="middle">Room, on</text> <text fill="currentColor" x="285" y="50" text-anchor="middle">a node</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="37" y1="84" x2="155" y2="84" marker-end="url(#alc-seq-arrow-n-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="-2" y="65" width="212.6" height="15" rx="3"></rect><text fill="currentColor" x="104.3" y="77" text-anchor="middle">join() on lobby, with the token</text></g> <g><rect stroke="currentColor" x="99.5" y="112" width="123" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="161" y="128" text-anchor="middle">lobby is asleep</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="160" x2="279" y2="160" marker-end="url(#alc-seq-arrow-n-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="153" y="141" width="140" height="15" rx="3"></rect><text fill="currentColor" x="223" y="153" text-anchor="middle">start on a free node</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="200" x2="279" y2="200" marker-end="url(#alc-seq-arrow-n-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="199.2" y="181" width="47.599999999999994" height="15" rx="3"></rect><text fill="currentColor" x="223" y="193" text-anchor="middle">join()</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="285" y1="240" x2="43" y2="240" marker-end="url(#alc-seq-arrow-n-zzqx6p)"></line><rect stroke="currentColor" fill="none" x="153.7" y="221" width="14.6" height="15" rx="3"></rect><text fill="currentColor" x="161" y="233" text-anchor="middle">4</text></g></svg>

The caller names the room and the gateway does the rest: it starts the room if it's asleep, then passes the call on. The caller never needs a route to the nodes.

## Get the class

A caller yields the class in its Construction phase, like any other binding:

```typescript
Effect.gen(function* () {
  const rooms = yield* Room;
  return {
    fetch: Effect.gen(function* () {
      // ...
    }),
  };
});
```

`rooms` is the class’s namespace: every room, addressed by name. Yielding it gives the caller the control plane’s address and a token to call it with. It works the same in a Cloudflare Worker, a Lambda, a container or a Hetzner Service, because the caller only speaks HTTP.

## Name an instance

```typescript
const lobby = rooms.getByName("lobby");
```

`getByName` doesn’t contact anything. It returns a stub for the lobby, and the first call through the stub reaches it. A name nobody has used before gets a fresh room.

## Call a method

```typescript
const visitors = yield* lobby.join();
```

The stub has the class’s methods, typed from it, so `join()` returns an `Effect<number>` because the room’s `join` does. Each call runs in the room, against the lobby’s state, wherever the lobby is running.

## Stream results

A method that returns a `Stream` sends its values as they’re produced, rather than all at the end. A room can stream its messages instead of returning them as one array:

```typescript
history: () =>
  Stream.fromIterableEffect(
    Effect.gen(function* () {
        "SELECT author, text FROM messages",
      );
      return yield* rows.toArray();
    }),
  ),
```

The caller consumes it like any other stream:

```typescript
yield* lobby.history().pipe(Stream.runForEach((message) => Effect.log(message.text)));
```

## Fail with a typed error

When a room says no, it fails with a tagged error. A full room refuses another visitor:

```typescript
export class RoomFull extends Data.TaggedError("RoomFull")<{ readonly limit: number }> {}
```

```typescript
join: () => (visitors >= 50 ? Effect.fail(new RoomFull({ limit: 50 })) : admit()),
```

The error crosses the wire with its tag and its fields, so the caller catches it by name:

```typescript
yield* lobby.join().pipe(
  Effect.catchTag("RoomFull", (error) => Effect.log(\`the lobby holds ${error.limit}\`)),
);
```

## Forward an HTTP request

A room with a `fetch` handler answers HTTP requests. A caller forwards the request it received:

```typescript
const request = yield* HttpServerRequest.HttpServerRequest;
return yield* lobby.fetch(request);
```

The room sees the request as it arrived, and its response goes back to the caller unchanged.

## Call one Durable Object from another

A Durable Object calls another the same way an Api does: it yields the class in its Construction phase. A `Greeter` that welcomes people into a room:

```typescript
export default class Greeter extends Rivet.DurableObject<Greeter>()(
  "Greeter",
  { main: import.meta.url },
  Effect.gen(function* () {
    const rooms = yield* Room;
    return Effect.gen(function* () {
      return {
        welcome: (room: string) =>
          rooms
            .getByName(room)
            .join()
            .pipe(Effect.map((visitors) => \`you're visitor ${visitors}\`)),
      };
    });
  }).pipe(Effect.provide(Default)),
) {}
```

The call goes through the control plane too, even when both run on the same pool.

## Keep the room’s code out of the caller

Importing `Room` brings its implementation into the caller’s bundle. To import only the declaration, split the class from its code. See [Durable Object](durable-objects.md#keep-its-code-out-of-its-callers).

Next, [RivetKit Actors](rivetkit.md): Actors written with Rivet’s own library, deployed as they are.
