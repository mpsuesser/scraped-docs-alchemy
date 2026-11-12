---
url: https://alchemy.run/modal/storage/queues
title: "Queues"
description: "Hand work between Workers, Modal sandboxes and Python functions with a Modal Queue."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **Queue** hands work from whoever puts it to whoever takes it: a Worker, a Sandbox, a Function or Python code. Items are JSON and expire 24 hours after they’re put.

## Partitions keep each user’s work apart

A Queue can split into **partitions**, each with its own items in order. A consumer takes from one partition and never sees another’s:

<svg viewBox="0 0 601.8 168" style="max-width: 692px" role="img" aria-label="The Worker puts each user's tasks in that user's partition. session-sam has taken lint, and test waits behind it; session-ana never sees either."><defs><marker id="alc-q-arrow-nzfm0g" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="8" y="71" width="100" height="46" rx="8"></rect><text fill="currentColor" x="58" y="91" text-anchor="middle">Api</text> <text fill="currentColor" x="58" y="107" text-anchor="middle">Worker</text> <line stroke-opacity="0.2" stroke="currentColor" x1="108" y1="94" x2="176" y2="94" marker-end="url(#alc-q-arrow-nzfm0g)"></line><text fill="currentColor" x="143" y="86" text-anchor="middle">put</text></g> <rect stroke="currentColor" fill="none" x="178" y="30" width="209.8" height="128" rx="10"></rect><text fill="currentColor" x="182" y="20">Jobs · Modal Queue</text> <text fill="currentColor" x="194" y="61">sam</text> <text fill="currentColor" x="194" y="117">ana</text> <g><path stroke="currentColor" fill="none" d="M 371.8 57 C 437.8 57, 427.8 53, 475.8 53" marker-end="url(#alc-q-arrow-nzfm0g)"></path><rect stroke="currentColor" fill="none" x="307.8" y="42" width="64" height="30" rx="5"></rect><text fill="currentColor" x="339.8" y="61" text-anchor="middle" opacity="0.75">lint</text> <text fill="currentColor" x="339.8" y="84" text-anchor="middle">taken</text></g> <g><rect stroke="currentColor" fill="none" x="229.8" y="42" width="64" height="30" rx="5"></rect><text fill="currentColor" x="261.8" y="61" text-anchor="middle" opacity="1">test</text></g> <g><path stroke="currentColor" fill="none" d="M 371.8 113 C 437.8 113, 427.8 135, 475.8 135" marker-end="url(#alc-q-arrow-nzfm0g)"></path><rect stroke="currentColor" fill="none" x="307.8" y="98" width="64" height="30" rx="5"></rect><text fill="currentColor" x="339.8" y="117" text-anchor="middle" opacity="0.75">build</text> <text fill="currentColor" x="339.8" y="140" text-anchor="middle">taken</text></g> <g><rect stroke="currentColor" fill="none" x="229.8" y="98" width="64" height="30" rx="5"></rect><text fill="currentColor" x="261.8" y="117" text-anchor="middle" opacity="1">deploy</text></g> <g><rect stroke="currentColor" fill="none" x="477.8" y="30" width="116" height="46" rx="8"></rect><text fill="currentColor" x="535.8" y="50" text-anchor="middle">session-sam</text> <text fill="currentColor" x="535.8" y="66" text-anchor="middle">Sandbox</text></g> <g><rect stroke="currentColor" fill="none" x="477.8" y="112" width="116" height="46" rx="8"></rect><text fill="currentColor" x="535.8" y="132" text-anchor="middle">session-ana</text> <text fill="currentColor" x="535.8" y="148" text-anchor="middle">Sandbox</text></g></svg> <svg viewBox="0 0 318 322" style="max-width: 366px" role="img" aria-label="The Worker puts each user's tasks in that user's partition. session-sam has taken lint, and test waits behind it; session-ana never sees either."><defs><marker id="alc-q-arrow-n-nzfm0g" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="109" y="0" width="100" height="46" rx="8"></rect><text fill="currentColor" x="159" y="20" text-anchor="middle">Api</text> <text fill="currentColor" x="159" y="36" text-anchor="middle">Worker</text> <line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="46" x2="159" y2="68" marker-end="url(#alc-q-arrow-n-nzfm0g)"></line><text fill="currentColor" x="167" y="64">put</text></g> <text fill="currentColor" x="4" y="82">Jobs · Modal Queue</text> <rect stroke="currentColor" fill="none" x="0" y="90" width="318" height="124" rx="10"></rect><text fill="currentColor" x="12" y="121">sam</text> <text fill="currentColor" x="12" y="177">ana</text> <path stroke="currentColor" fill="none" d="M 274 132 V 220 C 274 244, 96 234, 96 268" marker-end="url(#alc-q-arrow-n-nzfm0g)"></path><path stroke="currentColor" fill="none" d="M 274 188 V 220 C 274 244, 222 234, 222 268" marker-end="url(#alc-q-arrow-n-nzfm0g)"></path><g><rect stroke="currentColor" x="242" y="102" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="274" y="121" text-anchor="middle" opacity="0.75">lint</text> <text fill="currentColor" x="274" y="144" text-anchor="middle">taken</text></g> <g><rect stroke="currentColor" fill="none" x="168" y="102" width="64" height="30" rx="5"></rect><text fill="currentColor" x="200" y="121" text-anchor="middle" opacity="1">test</text></g> <g><rect stroke="currentColor" x="242" y="158" width="64" height="30" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="274" y="177" text-anchor="middle" opacity="0.75">build</text> <text fill="currentColor" x="274" y="200" text-anchor="middle">taken</text></g> <g><rect stroke="currentColor" fill="none" x="168" y="158" width="64" height="30" rx="5"></rect><text fill="currentColor" x="200" y="177" text-anchor="middle" opacity="1">deploy</text></g> <g><rect stroke="currentColor" fill="none" x="38" y="270" width="116" height="46" rx="8"></rect><text fill="currentColor" x="96" y="290" text-anchor="middle">session-sam</text> <text fill="currentColor" x="96" y="306" text-anchor="middle">Sandbox</text></g> <g><rect stroke="currentColor" fill="none" x="164" y="270" width="116" height="46" rx="8"></rect><text fill="currentColor" x="222" y="290" text-anchor="middle">session-ana</text> <text fill="currentColor" x="222" y="306" text-anchor="middle">Sandbox</text></g></svg>

The Worker puts each user's tasks in that user's partition. session-sam has taken lint, and test waits behind it; session-ana never sees either.

## Declare a Queue

```typescript
import * as Modal from "@alchemy.run/modal";

export const Jobs = Modal.Queue("Jobs", {});
```

## Put work on it

```typescript
const jobs = yield* Modal.WriteQueue(Jobs);
yield* jobs.put({ repo: "alchemy-run/alchemy", command: "pnpm test" });
```

Provide `Modal.WriteQueueHttp`.

## Take work off it

```typescript
const jobs = yield* Modal.ReadQueue(Jobs);
const batch = yield* jobs.take({ count: 10, timeout: "5 seconds" });
```

`take` removes items. `peek` reads them without removing.

## Put and take by partition

```typescript
yield* jobs.put(task, { partition: user });
const mine = yield* jobs.take({ partition: user });
```
