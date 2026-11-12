---
url: https://alchemy.run/rivet/durable-streams
title: "Durable Streams"
description: "Append-only logs with a replayable history, one per key, served by Rivet Services — declare one with its message Schema, append, read from an offset, follow it live, and choose its wire format as a Layer."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A **Durable Stream** is an append-only log with a durable, replayable history. Writers append messages to the end, and readers read from any offset or follow the end live. [Durable Streams](https://github.com/durable-streams/durable-streams) is an open protocol, and Rivet serves it from **Rivet Services**, a prebuilt node Rivet ships: each stream is its own Actor, with its messages kept by the control plane like any other Actor’s data.

Take a chat room’s messages. Each room has a stream, keyed by the room’s name. The room appends every message, a page loads the history from the start, and a live view follows new messages as they arrive:

<svg viewBox="0 0 625.8000000000001 128" style="max-width: 720px" role="img" aria-label="Messages stay in the order they were appended. A read returns them from an offset and says where to read next; a tail is caught up, and gets each new message as it's appended."><defs><marker id="alc-log-arrow-fyye63" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><text fill="currentColor" x="4" y="57">ChatLog/lobby</text> <g><rect stroke="currentColor" fill="none" x="118.2" y="38" width="80.4" height="30" rx="5"></rect><text fill="currentColor" x="158.4" y="57" text-anchor="middle">sam: hello</text> <text fill="currentColor" x="158.4" y="81" text-anchor="middle" opacity="0.7">0</text></g> <g><rect stroke="currentColor" fill="none" x="202.60000000000002" y="38" width="80.4" height="30" rx="5"></rect><text fill="currentColor" x="242.8" y="57" text-anchor="middle">ana: hi!</text><text fill="currentColor" x="242.8" y="81" text-anchor="middle" opacity="0.7">1</text></g> <g><rect stroke="currentColor" fill="none" x="287" y="38" width="80.4" height="30" rx="5"></rect><text fill="currentColor" x="327.2" y="57" text-anchor="middle">sam: lunch?</text><text fill="currentColor" x="327.2" y="81" text-anchor="middle" opacity="0.7">2</text></g> <g><rect stroke="currentColor" fill="none" x="371.40000000000003" y="38" width="80.4" height="30" rx="5"></rect><text fill="currentColor" x="411.6" y="57" text-anchor="middle">ana: yes</text> <text fill="currentColor" x="411.6" y="81" text-anchor="middle" opacity="0.7">3</text></g> <rect stroke="currentColor" fill="none" x="455.8" y="38" width="80.4" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="611.8000000000001" y1="53" x2="540.2" y2="53" marker-end="url(#alc-log-arrow-fyye63)"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="327.2" y1="22" x2="327.2" y2="34" marker-end="url(#alc-log-arrow-fyye63)"></line><text fill="currentColor" x="327.2" y="18" text-anchor="middle">read({ from })</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="496" y1="98" x2="496" y2="86" marker-end="url(#alc-log-arrow-fyye63)"></line><text fill="currentColor" x="496" y="110" text-anchor="middle">tail()</text></g></g> <text fill="currentColor" x="617.8000000000001" y="30" text-anchor="end">append</text></svg> <svg viewBox="0 0 238.8 232" style="max-width: 275px" role="img" aria-label="Messages stay in the order they were appended. A read returns them from an offset and says where to read next; a tail is caught up, and gets each new message as it's appended."><g><text fill="currentColor" x="24" y="12">ChatLog/lobby</text> <g><text fill="currentColor" x="18" y="41" text-anchor="end" opacity="0.7">0</text> <rect stroke="currentColor" fill="none" x="24" y="22" width="84.4" height="30" rx="5"></rect><text fill="currentColor" x="66.2" y="41" text-anchor="middle">sam: hello</text></g> <g><text fill="currentColor" x="18" y="77" text-anchor="end" opacity="0.7">1</text> <rect stroke="currentColor" fill="none" x="24" y="58" width="84.4" height="30" rx="5"></rect><text fill="currentColor" x="66.2" y="77" text-anchor="middle">ana: hi!</text></g><g><text fill="currentColor" x="18" y="113" text-anchor="end" opacity="0.7">2</text> <rect stroke="currentColor" fill="none" x="24" y="94" width="84.4" height="30" rx="5"></rect><text fill="currentColor" x="66.2" y="113" text-anchor="middle">sam: lunch?</text></g><g><text fill="currentColor" x="18" y="149" text-anchor="end" opacity="0.7">3</text> <rect stroke="currentColor" fill="none" x="24" y="130" width="84.4" height="30" rx="5"></rect><text fill="currentColor" x="66.2" y="149" text-anchor="middle">ana: yes</text></g> <rect stroke="currentColor" fill="none" x="24" y="166" width="84.4" height="30" rx="5"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="66.2" y1="216" x2="66.2" y2="198" marker-end="url(#alc-log-arrow-n-fyye63)"></line><text fill="currentColor" x="74.2" y="226">append</text> <g><line stroke-opacity="0.2" stroke="currentColor" x1="128.4" y1="109" x2="112.4" y2="109" marker-end="url(#alc-log-arrow-n-fyye63)"></line><text fill="currentColor" x="132.4" y="113">read({ from })</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="128.4" y1="181" x2="112.4" y2="181" marker-end="url(#alc-log-arrow-n-fyye63)"></line><text fill="currentColor" x="132.4" y="185">tail()</text></g></g><defs><marker id="alc-log-arrow-n-fyye63" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs></svg>

Messages stay in the order they were appended. A read returns them from an offset and says where to read next; a tail is caught up, and gets each new message as it's appended.

## Run Rivet Services

Streams are served by a Services pool next to your own pools. Give it the same control plane:

```typescript
import * as RivetDocker from "@alchemy.run/rivet/Docker";
import * as Layer from "effect/Layer";

export const Cluster = RivetDocker.SwarmControlPlane("Rivet");

export const Default = RivetDocker.SwarmPool("Default").pipe(Layer.provide(Cluster));

export const Services = RivetDocker.ServicesPool("Services").pipe(Layer.provide(Cluster));
```

Every cloud has one: `RivetAWS.ServicesPool`, `RivetGCP.ServicesPool`, `RivetHetzner.ServicesPool` and the rest. The Services node connects out to the control plane like any other node. Callers reach streams through the gateway, with the token.

## Declare a stream

A stream declaration names its messages’ Schema:

```typescript
import * as Rivet from "@alchemy.run/rivet";
import * as Schema from "effect/Schema";

export const Message = Schema.Struct({ user: Schema.String, text: Schema.String });

export class ChatLog extends Rivet.DurableStream<ChatLog>()("ChatLog", {
  schema: Message,
}) {}
```

One declaration holds many streams, one per key: `ChatLog` ’s stream for the lobby and its stream for the kitchen are separate logs. A stream is created by its first append.

## Write and read it

`yield* ChatLog` in any runtime gives a typed client. Provide the Services pool and a format with it:

```typescript
import * as Rivet from "@alchemy.run/rivet";
import * as StreamFormat from "@alchemy.run/rivet/StreamFormat";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { ChatLog } from "./chat-log.ts";
import { Default, Services } from "./cluster.ts";

export default class Room extends Rivet.DurableObject<Room>()(
  "Room",
  { main: import.meta.url },
  Effect.gen(function* () {
    const log = yield* ChatLog;
    return Effect.gen(function* () {
      const state = yield* Rivet.DurableObjectState;
      return {
        say: (user: string, text: string) => log.append(state.id.name, { user, text }),
        history: () => log.read(state.id.name),
      };
    });
  }).pipe(Effect.provide(Layer.mergeAll(Default, Services, StreamFormat.Json))),
) {}
```

`append` writes one message, and `appendAll` writes several in one request, in order. A message that doesn’t match the Schema fails before it’s sent.

`read` returns the messages from an offset, the start by default, and where to read next:

```typescript
const { messages, next, upToDate } = yield* log.read("lobby");
const later = yield* log.read("lobby", { from: next });
```

`from: "now"` skips the history. A read of a stream nobody appended to fails with `Rivet.DurableStreamNotFound`.

## Follow it live

`tail` is a `Stream` of every message from an offset, then each new one as it’s appended:

```typescript
yield* log.tail("lobby", { from: next }).pipe(
  Stream.runForEach((message) => Console.log(\`${message.user}: ${message.text}\`)),
);
```

It reconnects on its own when the connection drops, from the last offset it saw, and ends when the stream is closed.

## Choose a format

A stream’s wire format is a Layer, provided where the client is built.

`StreamFormat.Json` sends each message as one JSON value, as `application/json`. Rivet’s dashboard can read it.

`StreamFormat.Ndjson` writes one JSON value per line on a byte stream, as `application/x-ndjson`.

`StreamFormat.SchemaBinary()` writes Effect’s compact binary codec on a byte stream, as `application/octet-stream`.

A program only bundles the formats it provides. A stream whose format differs from the rest of the runtime’s takes its own Layer as `format`:

```typescript
import * as Rivet from "@alchemy.run/rivet";
import * as StreamFormat from "@alchemy.run/rivet/StreamFormat";
import * as Schema from "effect/Schema";

export const Sample = Schema.Struct({ at: Schema.Number, value: Schema.Number });

export class Telemetry extends Rivet.DurableStream<Telemetry>()("Telemetry", {
  schema: Sample,
  format: StreamFormat.SchemaBinary(),
}) {}
```

The format is fixed when a stream is created, so keep a stream’s format once it holds messages.

## Keep it, close it, delete it

A stream is kept until it’s deleted, unless the declaration sets its retention when it’s created:

```typescript
export class ChatLog extends Rivet.DurableStream<ChatLog>()("ChatLog", {
  schema: Message,
  ttl: "7 days",
}) {}
```

`ttl` deletes a stream after that long with no reads or writes; `expiresAt` deletes it at a fixed time.

`close` makes a stream final: reads and tails keep working, and appends fail with `Rivet.DurableStreamConflict`. `delete` removes it and its history:

```typescript
yield* log.close("lobby");
yield* log.delete("kitchen");
```

## Write exactly once

A writer that retries can append the same message twice. Give each append a producer, and a repeated one is written once:

```typescript
yield* log.append("lobby", message, { producer: { id: "room-lobby", epoch: 1, seq: 42 } });
```

`seq` goes up by one per append, and `epoch` goes up when the writer restarts. An append with an older epoch than the stream has seen fails with `Rivet.DurableStreamStaleProducer`.

## Where streams are reached

Callers reach every stream through the control plane’s gateway, with its token, never through the Services node: that node’s own port checks no token, and Services pools publish no port. See [Tokens and networks](tokens-and-networks.md#rivet-services-stays-private).

Services pools run `rivetdev/services:0.2.5` by default. A newer release goes in the pool’s `image`, after you’ve checked the streams you have still read back.

Every option, with its default, is in the reference: [DurableStream](https://alchemy.run/providers/rivet/durablestream).
