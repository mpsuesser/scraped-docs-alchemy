---
url: https://alchemy.run/domain/query
title: "Query"
description: "How a query serves a projection's store to clients across keys, typed, authorized by its handler, paged by cursor, and consistent with the caller's own writes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A query is a question a client asks across keys: “an account’s ledger, a page at a time”, “withdrawals over 1,000 across all accounts, largest first”. A [view](view.md) answers questions about one key. A [projection](projection.md) writes the store that can answer the others. A query serves that store.

Like a command, a query is a message. It declares its input and its result, the middleware that guards it, and the rejections it may return. Its Layer is one handler: it checks who is asking, then reads the store. An Api gives clients a typed method for it.

<svg viewBox="0 0 1188 112" style="max-width: 1366px" role="img" aria-label="The projection writes the store. The query reads it for a client, after judging who is asking."><defs><marker id="alc-dag-arrow-w3vbfg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-w3vbfg)"></path></g><g><path d="M 432 56 C 469 56, 469 56, 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-w3vbfg)"></path></g><g><path d="M 676 56 C 713 56, 713 56, 750 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-w3vbfg)"></path></g><g><path d="M 920 56 C 957 56, 957 56, 994 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-w3vbfg)"></path></g><g transform="translate(24, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">MoneyWithdrawn</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Ledger</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">projection</text></g></g> <g transform="translate(512, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">ledger table</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">D1</text></g> <g transform="translate(756, 24)"><g style="--hue:#e6d3a8"><path fill="none" d="M 12 0 H 158 Q 164 0 164 6 V 58 Q 164 64 158 64 H 12 L 0 32 Z"></path><g transform="translate(21.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M11 4a7 7 0 1 0 0 14a7 7 0 1 0 0-14z"></path><path stroke="currentColor" fill="none" d="M20 20l-4-4"></path></g><text fill="currentColor" x="41.300000000000004" y="28" dominant-baseline="middle">AccountLedger</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">query · handler</text></g></g> <g transform="translate(1000, 24)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Client</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">typed from the Api</text></g></g></svg> <svg viewBox="0 0 162 580" style="max-width: 186px" role="img" aria-label="The projection writes the store. The query reads it for a client, after judging who is asking."><defs><marker id="alc-dag-arrow-8cb4p6" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 81 74 C 81 101, 81 101, 81 128" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-8cb4p6)"></path></g><g><path d="M 81 198 C 81 225, 81 225, 81 252" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-8cb4p6)"></path></g><g><path d="M 81 322 C 81 349, 81 349, 81 376" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-8cb4p6)"></path></g><g><path d="M 81 446 C 81 473, 81 473, 81 500" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-8cb4p6)"></path></g><g transform="translate(10, 10)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="142" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="134" height="56" rx="5"></rect><g transform="translate(18.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="38.1" y="28" dominant-baseline="middle">Account a-1</text> <text fill="currentColor" x="71" y="46" text-anchor="middle" dominant-baseline="middle">MoneyWithdrawn</text></g></g> <g transform="translate(10, 134)"><g style="--hue:#3fb8af"><rect stroke="currentColor" fill="none" width="142" height="64" rx="8"></rect><g transform="translate(37.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 6c0-1.7 3.6-3 8-3s8 1.3 8 3-3.6 3-8 3-8-1.3-8-3z"></path><path stroke="currentColor" fill="none" d="M4 6v12c0 1.7 3.6 3 8 3s8-1.3 8-3V6"></path><path stroke="currentColor" fill="none" d="M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"></path></g><text fill="currentColor" x="57.6" y="28" dominant-baseline="middle">Ledger</text> <text fill="currentColor" x="71" y="46" text-anchor="middle" dominant-baseline="middle">projection</text></g></g> <g transform="translate(10, 258)"><rect stroke="currentColor" fill="none" width="142" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="71" y="28" text-anchor="middle" dominant-baseline="middle">ledger table</text> <text fill="currentColor" x="71" y="46" text-anchor="middle" dominant-baseline="middle">D1</text></g> <g transform="translate(10, 382)"><g style="--hue:#e6d3a8"><path fill="none" d="M 12 0 H 136 Q 142 0 142 6 V 58 Q 142 64 136 64 H 12 L 0 32 Z"></path><g transform="translate(10.300000000000004, 21) scale(0.5833333333333334)"><path fill="none" d="M11 4a7 7 0 1 0 0 14a7 7 0 1 0 0-14z"></path><path stroke="currentColor" fill="none" d="M20 20l-4-4"></path></g><text fill="currentColor" x="30.300000000000004" y="28" dominant-baseline="middle">AccountLedger</text> <text fill="currentColor" x="71" y="46" text-anchor="middle" dominant-baseline="middle">query · handler</text></g></g> <g transform="translate(10, 506)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="142" height="64" rx="8"></rect><g transform="translate(37.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="57.6" y="28" dominant-baseline="middle">Client</text> <text fill="currentColor" x="71" y="46" text-anchor="middle" dominant-baseline="middle">typed from the Api</text></g></g></svg>

AggregateQueryProjectionClient

The projection writes the store. The query reads it for a client, after judging who is asking.

## Declare it

The bank’s ledger is a table the `Ledger` projection writes. `AccountLedger` reads one account’s rows from it, a page at a time.

```typescript
export class AccountLedger extends Domain.Query("AccountLedger", {
  input: { account: AccountId },
  success: LedgerRow,
  paged: true,
  waitsFor: [Ledger],
  middleware: [Session],
  rejects: [NotOwner],
}) {}
```

- `input` holds the query’s fields, like a command’s. Filters, sort orders and scopes are fields too.
- `success` is the result. With `paged: true` it is one entry of a page, and the result is `{ entries, cursor }`. Leave `paged` out for a query with one result.
- `waitsFor` names the projections to wait on when a caller asks to read its own write. See [Read your own write](#read-your-own-write).
- `middleware` and `rejects` work as they do for commands and views. The handler fails with a rejection to refuse a caller.

`LedgerRow` is a schema, so the rows are checked and encoded on the way to the client:

```typescript
export const LedgerRow = Schema.Struct({
  id: Schema.String,
  accountId: Schema.String,
  kind: Schema.String,
  amount: Cents,
  balanceAfter: Cents,
  at: Schema.Number,
});
```

Add the query to the model’s `queries`, like a projection.

```typescript
projections: [Ledger, AccountCards],
streams: [AccountCard],
sagas: [ExecuteTransfer],
queries: [AccountLedger],
```

## Describe it

A query’s last argument is its [annotations](annotations.md). Its `description` is the [MCP](mcp.md) tool’s description, the [CLI](cli.md) subcommand’s help, and the operation’s description in the [OpenAPI](http-api.md) document.

```typescript
export class AccountLedger extends Domain.Query(
  "AccountLedger",
  {
    input: { account: AccountId },
    success: LedgerRow,
    paged: true,
    waitsFor: [Ledger],
    middleware: [Session],
    rejects: [NotOwner],
  },
  { description: "One account's ledger, a page at a time." },
) {}
```

## Implement it as a Layer

The handler lives in a Layer, built once with everything it needs: the store, and the `AccountSummary` view it checks the owner in. It receives the input and, for a paged query, the page asked for: `after`, the cursor of the previous page, and `take`, already capped by the Layer’s `take.max`.

```typescript
export const AccountLedgerLive = AccountLedger.toLayer(
  Effect.gen(function* () {
    const store = yield* LedgerStore;
    const summaries = yield* AccountSummary;
    return Effect.fn(function* ({ account }, page) {
      yield* Effect.annotateCurrentSpan("bank.account", account);
      const summary = yield* summaries.get(Account.ref(account)).pipe(Effect.orDie);
      yield* mustOwn(Option.getOrNull(summary)?.customerId ?? null);
      return yield* store.forAccount(account, page);
    });
  }),
  { take: { default: 50, max: 200 } },
);
```

The options tune how it runs, so they sit on the Layer, not the declaration. `take` limits page sizes: 50 rows when the client doesn’t ask, and never more than 200, whatever it asks for (50 and 500 by default). The handler records what of the input reaches a trace with `Effect.annotateCurrentSpan`, on the query’s own span; nothing else does.

It reads through the same `LedgerStore` port the projection writes through, so it sits next to the declaration in the domain package. The server provides the port with D1, and a story with memory. Merge the Layer into the model’s handlers, with the projections and policies.

The handler may also read what the query’s middleware provides, such as the current actor, to scope its store query by the caller. It may require `RuntimeContext`, as a database binding does.

## Ask it

An Api lists the queries it serves next to its commands and reads. Left out, `queries` serves every query of the model.

```typescript
export class CustomerApi extends Domain.Api("CustomerApi", {
  model: Bank,
  commands: ["RegisterCustomer", "DepositMoney", "WithdrawMoney"],
  views: ["AccountSummary"],
  queries: ["AccountLedger"],
}) {}
```

Its client calls the query as a method. Most calls are plain reads, a screen loading or the next page, and get what the store holds now:

```typescript
const page = yield* bank.AccountLedger({ account: accountId }, { take: 20 });
```

The result is decoded from the `success` schema. The error type lists the query’s rejections, its middleware’s errors, `ConsistencyTimeout` when it has `waitsFor`, and `RpcClientError`, so `Effect.catchTag` handles each.

A Foldkit app asks a query through a Command from `@alchemy.run/domain/Foldkit`. See [Integrate with Foldkit: Ask a query](foldkit.md#ask-a-query).

List the query’s name in `expose` to publish it beyond the typed client: as `QUERY /AccountLedger` with [`toHttpApi`](http-api.md#paths-and-methods), as `bank account-ledger --account a-1` with [`toCli`](cli.md#ask-a-query), or as a read-only tool with [`toToolkit`](mcp.md#tools-from-an-api).

## Page through it

Paging is by cursor. The handler returns `cursor: null` on the last page; otherwise the client passes the cursor back as `after`.

```typescript
const first = yield* bank.AccountLedger({ account: accountId }, { take: 20 });
if (first.cursor) {
  const next = yield* bank.AccountLedger({ account: accountId }, { after: first.cursor, take: 20 });
}
```

To read past one page, stream it. `entries` follows each cursor to the last page and yields every entry in order; `pages` yields the pages themselves. Both fetch a page only when the stream needs it, so `Stream.take` stops early:

```typescript
const latest = yield* bank.AccountLedger.entries({ account: accountId }, { take: 100 }).pipe(
  Stream.take(10),
  Stream.runCollect,
);
```

`after` starts the stream mid-way, and `atLeast` waits before the first page.

The cursor means nothing to the framework: the store chooses it. Page on the store’s sort key (keyset paging), so a row written between two pages is neither skipped nor repeated. The bank’s ledger sorts by time, then id:

```typescript
\`SELECT id, account_id AS accountId, kind, amount, balance_after AS balanceAfter, at FROM ledger WHERE account_id = ?${
  from ? " AND (at > ? OR (at = ? AND id > ?))" : ""
} ORDER BY at, id LIMIT ?\`,
```

## Sort it

Filters and sort orders are fields of the input. A query states exactly what a client may ask, so the store can index it and the handler can judge it. To let a client choose the order, declare the orders it may choose from:

```typescript
input: { account: AccountId, sort: Schema.Literals(["newest", "largest"]) },
```

Keyset paging works under any order, as long as the order is total: sort by the requested field, then by `id` to break ties, and put both values in the cursor. For “largest first”, the store reads the next page after the last row’s amount and id:

```sql
SELECT ... FROM ledger
WHERE account_id = ? AND (amount < ? OR (amount = ? AND id < ?))
ORDER BY amount DESC, id DESC LIMIT ?
```

Each order needs its own index (here on `account_id, amount, id`). That is why an order is one of a few declared values, not a column name the client sends.

A cursor only means something under the order that produced it, so make it carry the order, and give it a schema rather than joining values with a separator: ids such as `Account/a-1:3` may contain any separator. Encode the cursor as base64url JSON, so the client sees an opaque string:

```typescript
const LargestFirst = Schema.Struct({ order: Schema.Literal("largest"), amount: Cents, id: Schema.String });
const LargestFirstCursor = Schema.StringFromBase64Url.pipe(Schema.decodeTo(Schema.fromJsonString(LargestFirst)));

// the last row of a page → the next page's \`after\`
const cursorOf = (row: LedgerRow) =>
  Schema.encodeSync(LargestFirstCursor)({ order: "largest", amount: row.amount, id: row.id });
// \`after\` → where the next page starts: None for a cursor from another order, or not a cursor at all
const parseCursor = Schema.decodeUnknownOption(LargestFirstCursor);
```

When `parseCursor` returns `None`, fail with a declared rejection (`InvalidCursor`) rather than return a wrong page. A client that changes the order starts again from the first page.

## Decide who may ask, and which rows they see

The handler decides both. It checks the caller first: here, whether the caller owns the account or is an agent, through the same `mustOwn` rule the account’s views use. It fails with a rejection the query declares, which reaches the client typed.

It reads who is asking from what the middleware provides, and anything else it needs (a view’s client, a permissions store) from what its Layer resolved. When a caller passes `atLeast`, the query waits for its write before the handler runs.

It then scopes its store query by the caller, so a page holds only rows the caller may see, and comes back full:

```typescript
return Effect.fn(function* ({ min }, page) {
  const actor = yield* CurrentActor;
  return yield* store.largeWithdrawals({ min, branchId: actor.branchId }, page);
});
```

Like a view, a query is a security boundary: to show another audience other rows, declare another query.

## Read an aggregate’s current state

A client never reads an aggregate directly. A query can: its handler resolves the aggregate’s client and reads an instance’s state as of its latest commit, so it needs no `atLeast`. The handler checks the caller, and returns only what `success` declares:

```typescript
export class AccountBalance extends Domain.Query("AccountBalance", {
  input: { account: AccountId },
  success: Schema.Struct({ balance: Cents, frozen: Schema.Boolean }),
  middleware: [Session],
  rejects: [NotOwner, NotOpen],
}) {}

export const AccountBalanceLive = AccountBalance.toLayer(
  Effect.gen(function* () {
    const accounts = yield* Account;
    return Effect.fn(function* ({ account }) {
      const state = yield* accounts.get(Account.ref(account));
      if (state._tag !== "Open") return yield* new NotOpen();
      yield* mustOwn(state.customerId);
      return { balance: state.balance, frozen: state.frozen !== null };
    });
  }),
);
```

For its history, or to follow it live, read a [view](view.md) of it instead.

## Read your own write

A projection writes its store after the command commits, so a plain read can be a moment behind. Right after its own command, a client passes the command’s receipt as `atLeast`, and the query waits until that write is in the store before it reads:

```typescript
const receipt = yield* bank.WithdrawMoney(accountId, { amount: 30 }, mfa);
// receipt: { stream: "Account/a-1", version: 7 }
const page = yield* bank.AccountLedger({ account: accountId }, { atLeast: receipt });
```

`atLeast` is optional, and only for this moment. A read with no write behind it, a screen loading or the next page, leaves it out.

`waitsFor` tells the query which projections to wait on. The caller never names them, nor any key: it passes the receipt. The server reads that command’s events back from its aggregate and routes each through every projection in `waitsFor`, the same way the projection routes them when it handles them, to find the keys to wait on. Here, `WithdrawMoney` on `a-1` recorded `MoneyWithdrawn` at seq 7, and `Ledger` is keyed by account, so the query waits until `Ledger` has handled seq 7 for `a-1`.

The projections can be keyed differently. Each finds its own keys in the same events:

```typescript
export class BranchTotals extends Domain.Projection("BranchTotals", {
  from: [Account],
  key: BranchId,
  keyOf: { MoneyDeposited: (e) => e.branchId },
  on: [MoneyDeposited],
}) {}

export class BranchActivity extends Domain.Query("BranchActivity", {
  input: { branch: BranchId },
  success: ActivityRow,
  paged: true,
  waitsFor: [Ledger, BranchTotals],
}) {}

const receipt = yield* bank.DepositMoney(accountId, { amount: 100, branchId: "b-9" });
const page = yield* bank.BranchActivity({ branch: "b-9" }, { atLeast: receipt });
```

The deposit’s one event, `MoneyDeposited { branchId: "b-9" }`, lands on `Ledger` key `a-1` and on `BranchTotals` key `b-9`, and the query waits for both. A projection that doesn’t consume the receipt’s aggregate has nothing to wait for.

`waitsFor` takes views and streams too, for a handler that reads them. A view is waited on for the keys the command’s events reach, and a stream through the projection that writes it:

```typescript
export class AccountOverview extends Domain.Query("AccountOverview", {
  input: { account: AccountId },
  success: Overview,
  waitsFor: [AccountSummary, AccountCard],
  middleware: [Session],
  rejects: [NotOwner],
}) {}
```

The handler then reads `summaries.get(ref)` and `cards.peek(ref)` plainly: the engine already waited. A view waits only for an aggregate it reads directly, and a stream only for the aggregates its writer reads.

After commands to two aggregate instances, pass both receipts. The query waits for each, through every projection in `waitsFor`:

```typescript
const out = yield* bank.WithdrawMoney(checking, { amount: 30 }, mfa);
const into = yield* bank.DepositMoney(savings, { amount: 30 });
const total = yield* bank.CombinedBalance({ accounts: [checking, savings] }, { atLeast: [out, into] });
```

A wait longer than 10 seconds fails with `ConsistencyTimeout`, typed, so the client can retry or show that the data is behind.

Only the stores of the projections in `waitsFor` are covered. Without `waitsFor`, `atLeast` waits for nothing, and data the handler reads from anywhere else may still be behind. After two commands to the same aggregate instance, the later receipt is enough. A command a saga or a policy sends (here, the debit of a transfer) is not the client’s own write, so it has no receipt to pass: poll for it, or watch a view.

## Test it

A story asks a query with the context `Story.provide` set, as it sends commands: the handler runs, and the result goes through its schema as a client would get it. Give the story a store, with the other handlers.

```typescript
it.effect("the ledger is as private as the account: agents read it, other customers can't", () =>
  story(
    given(a1, opened(), new MoneyDeposited({ amount: 10_000, balanceAfter: 10_000 })),
    as(agent("support-7")),
    query(AccountLedger, { account: a1.id }, [{ kind: "MoneyDeposited", amount: 10_000 }]),
    as(alex),
    query(AccountLedger, { account: a1.id }),
    rejected(new NotOwner()),
  ),
);
```

Expect the rows in order, each a partial match; a partial result, for a query with one result; or a function over the result, to check a cursor. Without an expectation, the query must be rejected, and the next step asserts how. Projections run to the end after every `when`, so a story needs no `atLeast`.

To check the middleware and `atLeast` too, ask it through the Api’s `clientInProcess`. See [Testing an Api](testing-apis.md).

## Where it runs

A query runs where its Api runs: on Cloudflare, in the Worker, like a view’s `get`. It has no key, so no Durable Object owns it, and the store it reads is bound to the Worker. Its handler reads views from their Durable Objects, and `atLeast` reads checkpoints from the projection’s.

Each run is a `Domain.query AccountLedger` span with its outcome, the rows it returned, and what the handler annotated. See [Observability](observability.md).

Next, [Policy](policy.md) reacts to events by acting on the world.
