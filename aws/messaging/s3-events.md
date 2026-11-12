---
url: https://alchemy.run/aws/messaging/s3-events
title: "S3 events"
description: "How a Bucket notifies a Function when objects are created or deleted, what each notification carries, and how to process notifications as an Effect Stream."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A Bucket can tell you when something happens to its objects. Each time an object is created, deleted or restored, S3 sends a **notification** naming the Bucket, the key and the event. A Function that receives them can make a thumbnail of every uploaded image, index every new document or scan every file for viruses, without the uploader doing anything but upload.

## S3 invokes the Function

Notifications are pushed. S3 invokes the Function directly, with a batch of notifications, shortly after each change.

<svg viewBox="0 0 500 369" style="max-width: 575px" role="img" aria-label="S3 answers the upload first and invokes the Function shortly after. A notification can arrive twice, so writing the thumbnail again must be safe."><defs><marker id="alc-seq-arrow-ne8mop" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-ne8mop" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="365"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">uploader</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">your app</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="365"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Uploads</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">S3 Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="365"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Thumbnails</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">Lambda Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-ne8mop)"></line><rect stroke="currentColor" fill="none" x="99.30000000000001" y="66" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">PutObject photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="125" x2="88" y2="125" marker-end="url(#alc-seq-arrow-ne8mop)"></line><rect stroke="currentColor" fill="none" x="115.80000000000001" y="106" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="166" y="118" text-anchor="middle">200 OK, stored</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="165" x2="412" y2="165" marker-end="url(#alc-seq-arrow-ne8mop)"></line><rect stroke="currentColor" fill="none" x="277.2" y="146" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="334" y="158" text-anchor="middle">s3:ObjectCreated</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="205" x2="256" y2="205" marker-end="url(#alc-seq-arrow-ne8mop)"></line><rect stroke="currentColor" fill="none" x="267.3" y="186" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="334" y="198" text-anchor="middle">GetObject photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="245" x2="256" y2="245" marker-end="url(#alc-seq-arrow-ne8mop)"></line><rect stroke="currentColor" fill="none" x="244.2" y="226" width="179.6" height="15" rx="3"></rect><text fill="currentColor" x="334" y="238" text-anchor="middle">PutObject thumbs/photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="16" y1="293" x2="484" y2="293"></line><rect stroke="currentColor" fill="none" x="16" y="284" width="92.4" height="18" rx="4"></rect><text fill="currentColor" x="22" y="297">occasionally</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="325" x2="412" y2="325" marker-end="url(#alc-seq-arrow-warn-ne8mop)"></line><rect stroke="currentColor" fill="none" x="240.9" y="306" width="186.2" height="15" rx="3"></rect><text fill="currentColor" x="334" y="318" text-anchor="middle">the same notification again</text></g></svg> <svg viewBox="0 0 322 368" style="max-width: 370px" role="img" aria-label="S3 answers the upload first and invokes the Function shortly after. A notification can arrive twice, so writing the thumbnail again must be safe."><defs><marker id="alc-seq-arrow-n-ne8mop" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-ne8mop" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="49" y1="58" x2="49" y2="364"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="94" height="56" rx="8"></rect><text fill="currentColor" x="49" y="22" text-anchor="middle">uploader</text> <text fill="currentColor" x="49" y="37" text-anchor="middle">your app</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="58" x2="161" y2="364"></line><rect stroke="currentColor" fill="none" x="114" y="2" width="94" height="56" rx="8"></rect><text fill="currentColor" x="161" y="22" text-anchor="middle">Uploads</text> <text fill="currentColor" x="161" y="37" text-anchor="middle">S3 Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="273" y1="58" x2="273" y2="364"></line><rect stroke="currentColor" fill="none" x="226" y="2" width="94" height="56" rx="8"></rect><text fill="currentColor" x="273" y="22" text-anchor="middle">Thumbnails</text> <text fill="currentColor" x="273" y="37" text-anchor="middle">Lambda</text> <text fill="currentColor" x="273" y="50" text-anchor="middle">Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="49" y1="84" x2="155" y2="84" marker-end="url(#alc-seq-arrow-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="38.300000000000004" y="65" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="105" y="77" text-anchor="middle">PutObject photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="124" x2="55" y2="124" marker-end="url(#alc-seq-arrow-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="54.800000000000004" y="105" width="100.39999999999999" height="15" rx="3"></rect><text fill="currentColor" x="105" y="117" text-anchor="middle">200 OK, stored</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="164" x2="267" y2="164" marker-end="url(#alc-seq-arrow-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="160.2" y="145" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="217" y="157" text-anchor="middle">s3:ObjectCreated</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="273" y1="204" x2="167" y2="204" marker-end="url(#alc-seq-arrow-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="150.3" y="185" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="217" y="197" text-anchor="middle">GetObject photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="273" y1="244" x2="167" y2="244" marker-end="url(#alc-seq-arrow-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="127.19999999999999" y="225" width="179.6" height="15" rx="3"></rect><text fill="currentColor" x="217" y="237" text-anchor="middle">PutObject thumbs/photo.jpg</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="2" y1="292" x2="320" y2="292"></line><rect stroke="currentColor" fill="none" x="2" y="283" width="92.4" height="18" rx="4"></rect><text fill="currentColor" x="8" y="296">occasionally</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="324" x2="267" y2="324" marker-end="url(#alc-seq-arrow-warn-n-ne8mop)"></line><rect stroke="currentColor" fill="none" x="123.9" y="305" width="186.2" height="15" rx="3"></rect><text fill="currentColor" x="217" y="317" text-anchor="middle">the same notification again</text></g></svg>

S3 answers the upload first and invokes the Function shortly after. A notification can arrive twice, so writing the thumbnail again must be safe.

S3 delivers each notification at least once, and occasionally more than once, so processing the same key twice should be safe. Write the thumbnail under a key derived from the original’s, and a second run just replaces it.

Subscribe from the Function’s Construction phase. Notifications arrive as a `Stream`:

```typescript
export default class Thumbnails extends AWS.Lambda.Function<Thumbnails>()(
  "Thumbnails",
  { main: import.meta.url, functionUrl: false },
  Effect.gen(function* () {
    const uploads = yield* Uploads;
    yield* AWS.S3.consumeBucketEvents(uploads, (stream) =>
      stream.pipe(Stream.runForEach((event) => Console.log(\`${event.type}: ${event.key}\`))),
    );
    return {};
  }).pipe(Effect.provide(AWS.Lambda.BucketEventSource)),
) {}
```

Deploy adds the notification to the Bucket and the Lambda permission that lets S3 invoke the Function. Each notification carries its `type`, `bucket`, `key`, `size` and `eTag`.

## Choose which events

Without a filter, the Function receives every kind of notification: created, removed, restored, replicated. Name the ones you want:

```typescript
yield* AWS.S3.consumeBucketEvents(uploads, { events: ["s3:ObjectCreated:*"] }, (stream) =>
  stream.pipe(Stream.runForEach((event) => Console.log(event.key))),
);
```

The event names are checked against S3’s own list, so a typo fails to compile.

## Process each object

To read each new object, bind `S3.GetObject` to the same Bucket and run an Effect per notification:

```typescript
const getObject = yield* AWS.S3.GetObject(uploads);

yield* AWS.S3.consumeBucketEvents(uploads, { events: ["s3:ObjectCreated:*"] }, (stream) =>
  stream.pipe(
    Stream.mapEffect(
      (event) =>
        getObject({ Key: event.key }).pipe(
          Effect.flatMap((object) => makeThumbnail(event.key, object.Body!)),
          Effect.orDie,
        ),
      { concurrency: 4 },
    ),
    Stream.runDrain,
  ),
);
```

Provide `AWS.S3.GetObjectHttp` next to the event source. Here four objects are processed at a time. Leave out `concurrency` to take them one by one.

A batch is one invocation. The stream ends when the batch is done, and the next invocation starts a new one.

## Don’t trigger yourself

A Function that writes its output into the Bucket it watches will be notified about its own output. A thumbnail of an upload produces a new object, which produces a notification, which produces a thumbnail of the thumbnail.

Write outputs to a second Bucket, or keep inputs and outputs under separate prefixes and ignore the output prefix:

```typescript
stream.pipe(
  Stream.filter((event) => !event.key.startsWith("thumbnails/")),
  Stream.runForEach((event) => Console.log(event.key)),
)
```
