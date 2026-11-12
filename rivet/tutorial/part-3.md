---
url: https://alchemy.run/rivet/tutorial/part-3
title: "Part 3: Clear Idle Rooms"
description: "Use a Durable Object alarm to clear a room's messages an hour after the last one."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A room nobody uses should forget its messages. Each room has one alarm: a time at which Rivet calls the room again, whether or not anyone else does. Set it on every post, and clear the room when it fires.

## Pick the idle time

```typescript
import { Default } from "./cluster.ts";

const IDLE_MS = 60 * 60 * 1000;
```

## Set the alarm on every post

```typescript
post: (author: string, text: string) =>
  Effect.gen(function* () {
    yield* (yield* sql.exec(
      "INSERT INTO messages (author, text) VALUES (?, ?)",
      text,
    )).toArray();
    const now = yield* Effect.sync(() => Date.now());
    yield* state.storage.setAlarm(now + IDLE_MS);
  }),
```

`setAlarm` takes a time in milliseconds. Setting it again moves it, so the alarm keeps getting pushed back while people talk, and fires an hour after the last post.

## Clear the room when it fires

```typescript
messages: () =>
    Effect.gen(function* () {
        "SELECT author, text FROM messages",
      );
      return yield* rows.toArray();
    }),
  alarm: () =>
    Effect.gen(function* () {
      yield* (yield* sql.exec("DELETE FROM messages")).toArray();
    }),
};
```

`alarm` isn’t a method callers can run. Rivet calls it when the alarm comes due.

## Deploy

```sh
pnpm alchemy deploy
```

Post a message, then read it back an hour later. The list is empty. The visitor count stays, because it lives in key-value storage, not in the table.

You have rooms that keep a conversation while people talk, and forget it when they leave.
