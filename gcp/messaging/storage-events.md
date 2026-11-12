---
url: https://alchemy.run/gcp/messaging/storage-events
title: "Storage events"
description: "Run code when an object lands in a Cloud Storage bucket with GCP.Storage.consumeBucketEvents."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

`consumeBucketEvents` runs a handler when objects are written to or deleted from a Bucket. Uploads stay fast because the work happens in the background, on another service.

## Handle new uploads

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import * as Stream from "effect/Stream";
import { Uploads } from "./resources.ts";

export default class Indexer extends GCP.Function<Indexer>()(
  "Indexer",
  { main: import.meta.url },
  Effect.gen(function* () {
    const bucket = yield* Uploads;
    const uploads = yield* GCP.Storage.ReadBucket(bucket);

    yield* GCP.Storage.consumeBucketEvents(
      bucket,
      { eventTypes: ["OBJECT_FINALIZE"], prefix: "uploads/" },
      (events) =>
        events.pipe(
          Stream.runForEach((event) =>
            Effect.gen(function* () {
              const object = yield* uploads.get(event.object, {
                generation: event.generation,
              });
              if (object === undefined) return;
              yield* Effect.log(\`${event.object}: ${object.body.byteLength} bytes\`);
            }),
          ),
          Effect.orDie,
        ),
    );

    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }).pipe(
    Effect.provide([GCP.Storage.BucketEventSourceLive, GCP.Storage.ReadBucketHttp]),
    Effect.provide(GCP.Run.TopicEventSource),
  ),
) {}
```

`OBJECT_FINALIZE` fires when an object is written. `prefix` limits events to one folder.

Cloud Storage publishes each event to a Pub/Sub topic, and Pub/Sub pushes it to the service. That’s why the service provides `GCP.Run.TopicEventSource` as well.

<svg viewBox="0 0 668.8 289" style="max-width: 769px" role="img" aria-label="The upload returns as soon as the object is written. The notification travels through Pub/Sub to the Indexer, which reads the exact version the event names."><defs><marker id="alc-seq-arrow-256pbk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-256pbk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82.1" y1="59" x2="82.1" y2="285"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132.2" height="43" rx="8"></rect><text fill="currentColor" x="82.1" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="82.1" y="51" text-anchor="middle">Cloud Run service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250.29999999999998" y1="59" x2="250.29999999999998" y2="285"></line><rect stroke="currentColor" fill="none" x="184.2" y="16" width="132.2" height="43" rx="8"></rect><text fill="currentColor" x="250.29999999999998" y="36" text-anchor="middle">Uploads</text> <text fill="currentColor" x="250.29999999999998" y="51" text-anchor="middle">Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418.5" y1="59" x2="418.5" y2="285"></line><rect stroke="currentColor" fill="none" x="352.4" y="16" width="132.2" height="43" rx="8"></rect><text fill="currentColor" x="418.5" y="36" text-anchor="middle">notifications</text> <text fill="currentColor" x="418.5" y="51" text-anchor="middle">Pub/Sub topic</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586.6999999999999" y1="59" x2="586.6999999999999" y2="285"></line><rect stroke="currentColor" fill="none" x="520.5999999999999" y="16" width="132.2" height="43" rx="8"></rect><text fill="currentColor" x="586.6999999999999" y="36" text-anchor="middle">Indexer</text> <text fill="currentColor" x="586.6999999999999" y="51" text-anchor="middle">Cloud Run service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82.1" y1="85" x2="244.29999999999998" y2="85" marker-end="url(#alc-seq-arrow-256pbk)"></line><rect stroke="currentColor" fill="none" x="96.19999999999999" y="66" width="140" height="15" rx="3"></rect><text fill="currentColor" x="166.2" y="78" text-anchor="middle">put uploads/scan.pdf</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250.29999999999998" y1="125" x2="412.5" y2="125" marker-end="url(#alc-seq-arrow-256pbk)"></line><rect stroke="currentColor" fill="none" x="280.9" y="106" width="107" height="15" rx="3"></rect><text fill="currentColor" x="334.4" y="118" text-anchor="middle">OBJECT_FINALIZE</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418.5" y1="165" x2="580.6999999999999" y2="165" marker-end="url(#alc-seq-arrow-256pbk)"></line><rect stroke="currentColor" fill="none" x="455.7" y="146" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="502.59999999999997" y="158" text-anchor="middle">push (signed)</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586.6999999999999" y1="205" x2="256.29999999999995" y2="205" marker-end="url(#alc-seq-arrow-256pbk)"></line><rect stroke="currentColor" fill="none" x="338.59999999999997" y="186" width="159.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="418.49999999999994" y="198" text-anchor="middle">get(object, generation)</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586.6999999999999" y1="245" x2="424.5" y2="245" marker-end="url(#alc-seq-arrow-256pbk)"></line><rect stroke="currentColor" fill="none" x="442.5" y="226" width="120.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="502.59999999999997" y="238" text-anchor="middle">200: acknowledged</text></g></svg> <svg viewBox="0 0 322 288" style="max-width: 370px" role="img" aria-label="The upload returns as soon as the object is written. The notification travels through Pub/Sub to the Indexer, which reads the exact version the event names."><defs><marker id="alc-seq-arrow-n-256pbk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-256pbk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="58" x2="38.75" y2="284"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="38.75" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="38.75" y="37" text-anchor="middle">Cloud Run</text> <text fill="currentColor" x="38.75" y="50" text-anchor="middle">service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="58" x2="120.25" y2="284"></line><rect stroke="currentColor" fill="none" x="83.5" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="120.25" y="22" text-anchor="middle">Uploads</text> <text fill="currentColor" x="120.25" y="37" text-anchor="middle">Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="58" x2="201.75" y2="284"></line><rect stroke="currentColor" fill="none" x="165" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="201.75" y="22" text-anchor="middle">notifications</text> <text fill="currentColor" x="201.75" y="37" text-anchor="middle">Pub/Sub</text> <text fill="currentColor" x="201.75" y="50" text-anchor="middle">topic</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="58" x2="283.25" y2="284"></line><rect stroke="currentColor" fill="none" x="246.5" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="283.25" y="22" text-anchor="middle">Indexer</text> <text fill="currentColor" x="283.25" y="37" text-anchor="middle">Cloud Run</text> <text fill="currentColor" x="283.25" y="50" text-anchor="middle">service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="84" x2="114.25" y2="84" marker-end="url(#alc-seq-arrow-n-256pbk)"></line><rect stroke="currentColor" fill="none" x="9.5" y="65" width="140" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="77" text-anchor="middle">put uploads/scan.pdf</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="124" x2="195.75" y2="124" marker-end="url(#alc-seq-arrow-n-256pbk)"></line><rect stroke="currentColor" fill="none" x="107.5" y="105" width="107" height="15" rx="3"></rect><text fill="currentColor" x="161" y="117" text-anchor="middle">OBJECT_FINALIZE</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="164" x2="277.25" y2="164" marker-end="url(#alc-seq-arrow-n-256pbk)"></line><rect stroke="currentColor" fill="none" x="195.6" y="145" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="242.5" y="157" text-anchor="middle">push (signed)</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="204" x2="126.25" y2="204" marker-end="url(#alc-seq-arrow-n-256pbk)"></line><rect stroke="currentColor" fill="none" x="121.85000000000001" y="185" width="159.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="201.75" y="197" text-anchor="middle">get(object, generation)</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="244" x2="207.75" y2="244" marker-end="url(#alc-seq-arrow-n-256pbk)"></line><rect stroke="currentColor" fill="none" x="182.4" y="225" width="120.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="242.5" y="237" text-anchor="middle">200: acknowledged</text></g></svg>

The upload returns as soon as the object is written. The notification travels through Pub/Sub to the Indexer, which reads the exact version the event names.

## Read the exact version

Pass `event.generation` to `get`. If the object was overwritten or deleted since the event, you get `undefined` and a later event owns the work.

## Retries

A failed handler fails the delivery, and Pub/Sub sends the event again. Write results keyed by the object name so a second run overwrites the first.

## Example

[`examples/gcp-storage-uploads`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-storage-uploads) uploads through one service and indexes each file into Firestore from another.

## Where next

- [Cloud Storage](../data/storage.md) — buckets and their bindings.
- [Eventarc](eventarc.md) — events from other Google services.
