---
url: https://alchemy.run/cloudflare/ai/vectorize
title: "Vector search with Vectorize"
description: "Create a Vectorize index, bind it to a Worker with SearchIndex, and run filtered nearest-neighbor queries over vector embeddings."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Vectorize is Cloudflare’s globally distributed vector database. In alchemy it’s three pieces: a `Vectorize.Index` resource that holds the vectors, an optional `Vectorize.MetadataIndex` that makes a metadata property filterable, and a `Vectorize.SearchIndex` binding that gives a Worker a typed client to insert and query.

Docs · Vectorize index · 768 dimensions · cosinequery(embed(question), topK: 2)

| id | values | metadata |
| --- | --- | --- |
| doc-1#0 | \[0.12, -0.08, 0.31, …\] | { title: "Billing" } |
| doc-1#1 | \[0.09, -0.11, 0.27, …\] | { title: "Billing" } |
| doc-2#0 | \[-0.44, 0.20, 0.03, …\] | { title: "Deploys" } |
| doc-3#0 | \[0.10, -0.05, 0.29, …\] | { title: "Refunds" } |

Each vector is the embedding of a piece of text. A query returns the vectors closest to the question's embedding, with their ids and metadata.

## Create an index

Declare the index once at module scope so both the stack and the Worker can reference it:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Embeddings = Cloudflare.Vectorize.Index("Embeddings", {
  dimensions: 768,
  metric: "cosine",
});
```

`dimensions` must match the embedding model you’ll use (768 for `bge-base`, 1536 for `text-embedding-ada-002`, …) and `metric` is one of `cosine`, `euclidean`, or `dot-product`. A Vectorize index is immutable — changing dimensions, metric, preset, or description triggers a replacement, and the replacement is a fresh, empty index.

## Index a metadata property

To filter query results by a metadata property, that property needs a metadata index. Create it after resolving the parent index:

```typescript
const index = yield* Embeddings;

yield* Cloudflare.Vectorize.MetadataIndex("KindMeta", {
  indexName: index.indexName,
  propertyName: "kind",
  indexType: "string",
});
```

`indexType` is `string`, `number`, or `boolean`, matching the values stored under `propertyName`. Create metadata indexes **before** inserting vectors — only vectors inserted after the metadata index exists are filterable. Like the parent, a metadata index is immutable, and replacing the parent index replaces it too.

## Bind the index to a Worker

`Cloudflare.Vectorize.SearchIndex(index)` in the Worker’s Construction phase attaches the native `vectorize` binding at deploy time and resolves to an Effect-native client at runtime:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Embeddings } from "./embeddings.ts";

export default Cloudflare.Worker(
  "Search",
  { main: import.meta.url },
  Effect.gen(function* () {
    const index = yield* Embeddings;
    const vec = yield* Cloudflare.Vectorize.SearchIndex(index);

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        // routes below use \`vec\`
        return HttpServerResponse.text("ok");
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Vectorize.SearchIndexBinding)),
);
```

The client exposes `upsert`, `insert`, `query`, `queryById`, `getByIds`, `deleteByIds`, and `describe`, plus `raw` for direct access to the underlying runtime binding.

## Insert vectors

`upsert` writes vectors, replacing any with matching ids (`insert` instead fails on duplicate ids):

```typescript
const mutation = yield* vec.upsert([
  { id: "doc-1", values: embedding, metadata: { kind: "article" } },
  { id: "doc-2", values: other, metadata: { kind: "note" } },
]);
// mutation.mutationId identifies the async write
```

Each `values` array must have exactly the index’s `dimensions` entries. `metadata` is an arbitrary JSON object stored alongside the vector — properties covered by a metadata index become filterable.

## Query nearest neighbors

`query` finds the vectors closest to the one you pass:

```typescript
const results = yield* vec.query(queryEmbedding, {
  topK: 5,
  returnMetadata: "all",
});
// results.matches: [{ id, score, metadata }, ...] best match first
```

`topK` caps the number of matches, and `returnMetadata: "all"` includes each match’s stored metadata in the response.

## Filter by metadata

Pass a `filter` to restrict matches to vectors whose metadata satisfies it — this is what the metadata index enables:

```typescript
const articles = yield* vec.query(queryEmbedding, {
  topK: 5,
  returnMetadata: "all",
  filter: { kind: { $eq: "article" } },
});
```

Only vectors with `kind === "article"` are considered. A property without a metadata index cannot be used in a `filter`, so declare the `MetadataIndex` up front.

## Deploy it

The stack ties it together — index, metadata index, then the Worker that binds them:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import { Embeddings } from "./src/embeddings.ts";
import Worker from "./src/worker.ts";

export default Alchemy.Stack(
  "VectorSearch",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const index = yield* Embeddings;
    yield* Cloudflare.Vectorize.MetadataIndex("KindMeta", {
      indexName: index.indexName,
      propertyName: "kind",
      indexType: "string",
    });
    const worker = yield* Worker;
    return { url: worker.url.as<string>() };
  }),
);
```

```sh
pnpm alchemy deploy
```

## Where next

- [AI Search](ai-search.md) — managed RAG that handles embedding and indexing for you; reach for Vectorize when you want control over the vectors themselves.
- [Workers](../compute/workers.md) — the two-phase Worker model the `SearchIndex` binding lives inside.
- Reference: [Index](https://alchemy.run/providers/cloudflare/vectorize#index), [MetadataIndex](https://alchemy.run/providers/cloudflare/vectorize#metadataindex), [SearchIndex](https://alchemy.run/providers/cloudflare/vectorize#searchindex)
