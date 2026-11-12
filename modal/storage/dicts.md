---
url: https://alchemy.run/modal/storage/dicts
title: "Dicts"
description: "Share key-value data between Modal sandboxes, functions and Workers."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **Dict** is a key-value store in an environment. Functions, Servers, Sandboxes, Python code and your Workers read and write the same entries, so it’s where they share small results and state.

## Entries are JSON, and expire

Keys and values are JSON, so Python reads them with `json.loads`. An entry expires 7 days after its last write:

Results · Modal Dictget("last-run")

| key | value |
| --- | --- |
| last-run | { ok: true } |
| a | 1 |
| b | 2 |

Each entry is one JSON key and one JSON value. get reads one entry by its key.

## Declare a Dict

```typescript
import * as Modal from "@alchemy.run/modal";

export const Results = Modal.Dict("Results", {});
```

## Bind it in a runtime

```typescript
const results = yield* Modal.ReadWriteDict(Results);
```

Provide `Modal.ReadWriteDictHttp`. Use `ReadDict` or `WriteDict` for one direction.

## Write entries

```typescript
yield* results.put("last-run", { ok: true });
yield* results.update([["a", 1], ["b", 2]]);
```

## Read entries

```typescript
const last = yield* results.get<{ ok: boolean }>("last-run");
const all = yield* results.entries();
```

## Remove entries

```typescript
const popped = yield* results.pop("a");
yield* results.clear();
```

`pop` returns the value it removed.
