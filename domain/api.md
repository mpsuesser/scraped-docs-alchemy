---
url: https://alchemy.run/domain/api
title: "Api"
description: "How an Api publishes a model to one kind of client, how it's served, and how a typed client calls it."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

An Api is the interface a model publishes. It lists what one kind of client may reach: the commands it may send, and the views, streams and aggregates it may read. Everything else in the model stays private.

An Api has no handlers of its own. Aggregates already decide commands and views already answer reads, so an Api only chooses which of them to expose. This chapter starts with what it means to publish a model, then covers what an Api can expose, declaring one, serving it, and calling it.

## Publish the model, keep the rest private

A bounded context is meant to be used: by a web app, by a support console, by another context. DDD calls the interface it offers them a **published language**. It’s the part of the model other people may depend on, and it’s chosen on purpose.

Publishing a slice of the model, rather than all of it, is what keeps the model free to change. Clients depend on `DepositMoney` and `AccountSummary`. They don’t depend on how `Account` stores its state, which policies exist, or which projection feeds which table. You can rework all of those without breaking a client, as long as what you published still works.

<svg viewBox="0 0 700 352" role="img" aria-label="Clients reach what the Api lists. Everything else stays inside the model."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 88 L 350 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 296 L 506 296" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Customer app</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">a client</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CustomerApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the published part</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">DepositMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">published</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">published</text></g></g> <g transform="translate(512, 264)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">OpenAccount</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">internal</text></g></g> <g transform="translate(268, 264)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">a policy</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">internal</text></g></g></svg>

CommandViewPolicyApiClient

Clients reach what the Api lists. Everything else stays inside the model.

It’s also the first line of the model’s security. What isn’t published can’t be reached from outside at all.

## What an Api can expose

An Api can expose four kinds of entry. Commands are what a client sends to change something. Views, streams and aggregates are what a client reads.

Exposing an entry doesn’t bypass its security. Each entry declares its own middleware and checks, and they run on every call that comes through the Api.

<svg viewBox="0 0 700 472" role="img" aria-label="One Api, four kinds of entry. Each is guarded by its own middleware and rules."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 176 L 262 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 144 L 350 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 208 L 350 296 L 506 296" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 208 L 350 416 L 506 416" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 144)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Client</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">typed from the Api</text></g></g> <g transform="translate(268, 144)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CustomerApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Domain.Api</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">DepositMoney</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">command · send</text></g></g> <g transform="translate(512, 144)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(17.400000000000006, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="37.400000000000006" y="28" dominant-baseline="middle">AccountSummary</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">view · get, watch</text></g></g> <g transform="translate(512, 264)"><g style="--hue:#9aa7c7"><rect stroke="currentColor" fill="none" x="8" y="8" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M3 7h11"></path><path stroke="currentColor" fill="none" d="M3 12h17"></path><path stroke="currentColor" fill="none" d="M3 17h11"></path><path stroke="currentColor" fill="none" d="M17 9l3 3-3 3"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">AccountCard</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">stream · peek, watch</text></g></g> <g transform="translate(512, 384)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Account</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">aggregate · get, events</text></g></g></svg>

CommandAggregateViewStreamApiClient

One Api, four kinds of entry. Each is guarded by its own middleware and rules.

Policies, projections and external systems are never on an Api. They run inside the model, and a client reaches what they produce through the entries above: a projection’s [stream](stream.md), a view a policy feeds, or a store you serve yourself.

### Commands

A command becomes a method that takes its target and input together, and returns the receipt with the reply. Its middleware runs first, and its aggregate’s `decide` has the final say.

```typescript
const { reply } = yield* bank.DepositMoney(accountId, { amount: 500 });
```

### Views

A view becomes a namespace with five reads: `get` for the current state, `watch` for the state and every change, each with its cursor, `events` for its emitted events (history, a `CaughtUp` frame, then live), `list` for a page of them, and `snapshot` for the state with the cursor it is valid at. Streams and aggregates send the same shapes; see [Know when a live read is current](view.md#know-when-a-live-read-is-current). `get`, `list` and `snapshot` also take `atLeast`, to read your own write, and fail with `ConsistencyTimeout` if the view hasn’t applied it within 10 seconds. Its middleware and `validate` decide who may read which key, on every read and on every change while a client watches.

```typescript
const summary = yield* bank.AccountSummary.get(accountId);
const summaries = bank.AccountSummary.watch(accountId);
const page = yield* bank.Statement.list(accountId, { order: "desc", take: 10 });
```

### Streams

A stream becomes a namespace like a view’s, with `peek` in place of `get`: the latest entry is the current value. It’s guarded the same way, by its middleware and `validate`, judged against the latest entry. See [Stream](stream.md).

```typescript
const card = yield* bank.AccountCard.peek(accountId);
const cards = bank.AccountCard.watch(accountId);
```

### Aggregates

An aggregate can be read directly too: its current state with `get` and `watch`, and its committed events with `events`. It’s only allowed on an Api if it declares a `read` contract, because its state is otherwise private to its rules. The contract names the middleware and the check every read must pass.

```typescript
read: {
  middleware: [Session],
  rejects: [NotOwner],
  validate: Effect.fn(function* (_request, s) {
    yield* mustOwn(s._tag === "Open" ? s.customerId : null);
  }),
},
```

```typescript
const account = yield* bank.Account.get(accountId);
const history = bank.Account.events(accountId, { after: 0 });
```

Reading an aggregate directly is handy for an account’s own screen. For anything shaped for a question, like a statement or a dashboard, prefer a view: it keeps the aggregate’s state free to change with its rules.

## List what a client may reach

The tutorial’s bank exposes registration, deposits, withdrawals and the account summary.

```typescript
export class BankApi extends Domain.Api("BankApi", {
  model: Bank,
  commands: ["RegisterCustomer", "DepositMoney", "WithdrawMoney"],
  views: ["AccountSummary"],
}) {}
```

`model` is the model the Api is over. `commands`, `views`, `streams` and `aggregates` name what of it a client may reach, and naming something the model doesn’t hold is a type error. A kind you leave out is served whole, so `Domain.Api("BankApi", { model: Bank })` serves every command, view and stream, and every aggregate that declares `read`. `[]` serves none of a kind, and naming an aggregate without `read` is a type error.

Leaving something out is how you keep it internal. `OpenAccount` isn’t listed, because only the `MainAccountProvisioning` policy opens accounts. No client can send it, however it’s built.

## Give each kind of client its own Api

Different clients need different surfaces. In the example bank, customers and support agents use the same model through two Apis.

```typescript
export class SupportApi extends Domain.Api("SupportApi", {
  model: Bank,
  commands: ["FreezeAccount", "ChangeOwner", "ApproveTransfer"],
  views: ["AccountSummary", "AccountActivity"],
  streams: [],
  aggregates: [],
}) {}
```

`FreezeAccount` and `ChangeOwner` are on the support Api and not the customer one. `AccountSummary` is on both.

<svg viewBox="0 0 700 204" role="img" aria-label="One model, published to two kinds of client."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 C 225 56, 225 56, 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 188 148 C 225 148, 225 148, 262 148" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 C 469 56, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 148 C 469 148, 469 102, 506 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(25.200000000000003, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="45.2" y="28" dominant-baseline="middle">Customer app</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">web app</text></g></g> <g transform="translate(24, 116)"><g style="--hue:#c3ccd6"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(13.5, 21) scale(0.5833333333333334)"><path fill="none" d="M4 5h16v11H4z"></path><path stroke="currentColor" fill="none" d="M2 19h20"></path></g><text fill="currentColor" x="33.5" y="28" dominant-baseline="middle">Support console</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">internal tool</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(29.1, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="49.1" y="28" dominant-baseline="middle">CustomerApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">for customers</text></g></g> <g transform="translate(268, 116)"><g style="--hue:#ece8df"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3a9 9 0 1 0 0 18a9 9 0 1 0 0-18z"></path><path stroke="currentColor" fill="none" d="M3 12h18"></path><path stroke="currentColor" fill="none" d="M12 3c3 3 3 15 0 18"></path><path stroke="currentColor" fill="none" d="M12 3c-3 3-3 15 0 18"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">SupportApi</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">for support</text></g></g> <g transform="translate(512, 70)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">Bank</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">model</text></g></svg>

ApiClient

One model, published to two kinds of client.

Being on an Api doesn’t mean anyone may use an entry for anything. Each entry still guards itself. A command’s middleware establishes who’s calling and its `decide` checks whether they may; a view’s middleware and `validate` decide what they may see. So `AccountSummary` can sit on both Apis, and a customer still reads only their own accounts.

## Serve it

An Api is served from a Worker. The Api class is an Effect: `yield* BankApi` gives you its `fetch`, and the Worker returns it as its own. This is the tutorial’s bank:

```typescript
const BankLive = SessionLive.pipe(
  Layer.provideMerge(Bank.layer(BankHandlers)),
  Layer.provide(DomainCloudflare.layerDurableObjects()),
);

export default class BankWorker extends Cloudflare.Worker<BankWorker>()(
  "BankWorker",
  { main: import.meta.url },
  Effect.gen(function* () {
    const api = yield* BankApi;
    return { fetch: api.fetch };
  }).pipe(Effect.provide(BankLive)),
) {}
```

`BankLive` is everything the handler needs: the model hosted on Durable Objects, with its policies and projections, and an implementation of every middleware the Api’s entries declare, here the session check. Deploying the Worker deploys the Durable Objects with it.

Commands, `get`, `peek` and `list` are plain requests. Live reads, `watch` and `events`, are upgraded to WebSockets connected to the Durable Object that owns the key, which pushes each change as it commits.

A Worker can serve several Apis by routing on the path. The example bank serves customers on `/api` and support agents on `/support`:

```typescript
if (path.startsWith("/api")) return yield* customerApi.fetch;
if (path.startsWith("/support")) return yield* supportApi.fetch;
```

See [Cloudflare Durable Objects](cloudflare.md) for how the model is hosted.

## Call it with a typed client

`Api.client` builds a client typed from the Api’s entries. Each command is a method that takes the target, the input, and optional call options like `headers`. Each view or aggregate is a namespace with its reads.

```typescript
export const depositAndWatch = Effect.gen(function* () {
  const bank = yield* CustomerApi.client;
  const accountId = AccountId.make("c-1-main");
  const { reply } = yield* bank.DepositMoney(accountId, { amount: 1_000 });
  yield* Effect.log(\`balance ${reply.balance}\`);
  yield* bank.AccountSummary.watch(accountId).pipe(
    Stream.take(2),
    Stream.runForEach(({ state }) => Effect.log(\`balance is now ${state?.balance}\`)),
  );
});
```

## Choose a protocol

The client needs to know how to reach the server. `Domain.protocolWebSocket(url)` sends commands and one-off reads over HTTP and opens a WebSocket for each live read; if the connection drops, the client reconnects and resumes from the last change it saw. `Domain.protocolHttp(url)` uses HTTP for everything, streaming live reads as long-running responses.

Each protocol also has a shortcut on the Api that provides it:

```typescript
const bank = yield* CustomerApi.clientWebSocket(url);
// is short for
// CustomerApi.client.pipe(Effect.provide(Domain.protocolWebSocket(url)))
```

## Handle what can go wrong

Every call’s error type says exactly what can fail. It’s one of the rejections the entry declares, like `InsufficientFunds`; an error from its middleware, like `Unauthorized`; or an `RpcClientError` when the server can’t be reached. Because each is a tagged error, `Effect.catchTag` handles the cases you expect and leaves the rest to propagate.

```typescript
export const withdraw = Effect.fn(function* (bank: CustomerClient, accountId: AccountId) {
  const receipt = yield* bank.WithdrawMoney(accountId, { amount: 2_000 }, mfa).pipe(
    Effect.catchTag("InsufficientFunds", (r) => Effect.fail(\`only ${r.balance} available\`)),
  );
  return receipt.reply.balance;
});
```

## Read an Api’s commands and rejections

`CustomerApi.commands` lists the Api’s command classes, in order. `CustomerApi.rejections` is one Schema for every rejection they declare, so a UI message can hold a refusal as data:

```typescript
const Refused = Schema.TaggedStruct("Refused", {
  commandId: Schema.String,
  rejection: CustomerApi.rejections, // InsufficientFunds | NotOwner | ...
});
```

`Domain.Api.Any` is the type every Api fits. Use it for helpers you write once for all your Apis:

```typescript
const actionsOf = <A extends Domain.Api.Any>(api: A) =>
  api.commands.map((command) => ({ tag: command.tag, rejects: command.rejects }));
```

## Provide the client to an app

An app usually wants one client for its whole lifetime. `CustomerApi.Client` is the client as a service, and `CustomerApi.layerClient` provides it. The app’s entry point provides it once, with the protocol to use:

```typescript
return CustomerApi.layerClient.pipe(
  Layer.provide(Domain.protocolWebSocket(\`${location.origin}/api\`)),
  Layer.provide(Socket.layerWebSocketConstructorGlobal),
);
```

Anything that sends a command asks for the client, and doesn’t care how it’s connected:

```typescript
const bank = yield* CustomerApi.Client;
```

See [Integrate with foldkit](foldkit.md) for a full browser app.

## Call it in process

Code in the same process as the model, like a webhook route, a scheduled job or a script, calls the Api with `clientInProcess`. It’s the same typed client over `Domain.protocolInProcess`, which serves the Api right there: nothing goes over the network. Every entry’s middleware still runs, from the middleware Layers in scope.

Often that code has already checked who is calling, for example by verifying a webhook signature. Then it provides a middleware Layer that trusts that caller, to this client only. The Api the Worker serves keeps its real middleware.

```typescript
const asSystem = Layer.succeed(Session, (effect) =>
  Effect.provideService(effect, CurrentActor, system("payments-webhook")),
);

export const depositFromWebhook = Effect.fn(function* (accountId: AccountId, amount: number) {
  const bank = yield* CustomerApi.clientInProcess.pipe(Effect.provide(asSystem));
  return yield* bank.DepositMoney(accountId, { amount });
}, Effect.scoped);
```

The caller’s `decide` and `validate` read the `CurrentActor` the trusted Layer provides, exactly as they read the one `SessionLive` provides.

`clientInProcess` needs a `Scope`. In a Worker, build it per request, like the served Api: Workers forbid sharing I/O across requests. In a long-lived process, `CustomerApi.layerClientInProcess` provides `CustomerApi.Client` in process for the life of the Layer, so code that asks for the client runs the same in process or over the network.

Tests use the same client. To test an Api end to end, including its middleware and every rejection, see [Testing an Api](testing-apis.md).

To serve every call as a plain HTTP JSON API with an OpenAPI document, see [Plain HTTP and OpenAPI](http-api.md).

To give an agent the Api’s calls as tools, in the app or over MCP, see [MCP](mcp.md).

To run the Api’s calls from a terminal, see [CLI](cli.md).

Next, [Story](story.md): how to state each rule as an example, and run it.
