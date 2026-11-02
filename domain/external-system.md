---
url: https://alchemy.run/domain/external-system
title: "External System"
description: "Declare the outside systems your model calls in the model's own language, implement them with Layers, and turn their events into commands."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

An external system is a service outside the model that its policies call, such as a payment processor or a fraud scorer. The model declares the calls it makes, in its own words, with their arguments and results, and nothing about the vendor behind them. A Layer supplies the vendor, so production, tests and stories can each use a different one.

This chapter starts with why the model keeps outside systems at arm’s length, then declares one, implements it, calls it, and turns events from outside into commands.

## Keep the outside world at the edge

Every vendor has its own language. A payment processor talks about payment intents, charges, balance transactions and webhooks. A fraud vendor talks about risk signals and decision codes. None of those words belong to your model. If they leak into the model, the model starts speaking two languages, and changing vendor means rewriting rules.

DDD’s answer is a translation layer at the edge, sometimes called an **anti-corruption layer**. The model says what it needs in its own terms: “refund this settlement”, “score this withdrawal”. A small adapter outside the model turns that into whatever the vendor’s API wants, and turns the vendor’s answer back into the model’s terms.

<svg viewBox="0 0 700 232" role="img" aria-label="The model depends on the declaration. Production and tests plug in different implementations."><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 350 88 L 350 176 L 506 176" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#a78bdb"><path fill="none" d="M 9 0 H 155 L 164 9 V 55 L 155 64 H 9 L 0 55 V 9 Z"></path><g transform="translate(9.599999999999994, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M12 3v18"></path><path stroke="currentColor" fill="none" d="M6 21h12"></path><path stroke="currentColor" fill="none" d="M5 7h14"></path><path fill="none" d="M5 7l-3 7a3 3 0 0 0 6 0z"></path><path fill="none" d="M19 7l-3 7a3 3 0 0 0 6 0z"></path></g><text fill="currentColor" x="29.599999999999994" y="28" dominant-baseline="middle">SettlementRefund</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">calls refund</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#d97aa8"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M9 2v6"></path><path stroke="currentColor" fill="none" d="M15 2v6"></path><path fill="none" d="M6 8h12v4a6 6 0 0 1-12 0z"></path><path stroke="currentColor" fill="none" d="M12 18v4"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Payments</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">in the bank's words</text></g></g> <g transform="translate(512, 24)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">StripePayments</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">Layer · production</text></g> <g transform="translate(512, 144)"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="82" y="28" text-anchor="middle" dominant-baseline="middle">Story</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">you answer each call</text></g></svg>

PolicyExternal system

The model depends on the declaration. Production and tests plug in different implementations.

A `Domain.ExternalSystem` is that edge, written down. It names the calls the model makes and nothing else. The example bank declares two: `FraudCheck` and `Payments`.

It works in the other direction too. News from outside, like a webhook saying a payment settled, crosses the same edge and becomes a command in the model’s language.

## Declare an external system

A `Domain.ExternalSystem` lists the calls you make to one outside system, with their arguments and results. It’s an Effect service, like a `Context.Service`: policies `yield*` it, and a Layer implements it.

```typescript
export class Payments extends Domain.ExternalSystem("Payments", {
  refund: {
    args: { settlementId: Schema.String, amount: Cents },
    success: { refundId: Schema.String },
  },
}) {}
```

Name each call after what the model wants done, not after the vendor endpoint that does it. `refund` is a request in the model’s own language. `createRefundForPaymentIntent` would be Stripe’s language leaking in.

Keep the arguments and results to what the model uses. The fraud check returns a risk score, because that’s all the fraud policy decides on; the dozens of other fields the vendor sends stay in the adapter.

```typescript
export class FraudCheck extends Domain.ExternalSystem("FraudCheck", {
  score: { args: { accountId: AccountId, amount: Cents }, success: { risk: Schema.Number } },
}) {}
```

## Declare its errors

When the vendor can refuse, declare the error with `error:`. The call’s Effect then fails with that type, so a policy handles it like any typed error.

```typescript
class RefundDeclined extends Domain.Rejection("RefundDeclined", { reason: Schema.String }) {}

export class Payments extends Domain.ExternalSystem("Payments", {
  refund: {
    args: { settlementId: Schema.String, amount: Cents },
    success: { refundId: Schema.String },
    error: RefundDeclined,
  },
}) {}
```

When the vendor can refuse in several ways, list each error. The call fails with any of them, and a policy handles each one by its tag:

```typescript
class CardFrozen extends Domain.Rejection("CardFrozen") {}

export class Payments extends Domain.ExternalSystem("Payments", {
  refund: {
    args: { settlementId: Schema.String, amount: Cents },
    success: { refundId: Schema.String },
    error: [RefundDeclined, CardFrozen],
  },
}) {}
```

A `Schema.Union([RefundDeclined, CardFrozen])` declares the same errors.

Declare the refusals the model has an answer for, in its own words. A declined refund is a business outcome a policy can act on. A timeout or a 500 from the vendor isn’t; it’s a failure, and the policy that made the call runs again.

## Implement it with a Layer

An adapter is a Layer that implements every call. It’s the only code that knows the vendor, so swapping vendor, or using a fake, never touches the model. To prove a fake answers like the vendor, [run the same tests against both](testing-models.md#run-the-same-tests-against-every-adapter).

```typescript
export const StripePayments = Payments.toLayer(
  Effect.succeed({
    refund: ({ settlementId }) => Effect.succeed({ refundId: \`re_${settlementId}\` }),
  }),
);
```

This one is a stand-in. A production adapter calls the vendor’s HTTP API, maps its response onto `{ refundId }`, and maps its refusals onto the declared errors.

Adapters live in the server package, next to the platform, because they reach the network. The domain package only holds the declaration. See [Project layout](project-layout.md).

## Calls are checked against the schemas

Every call through a `toLayer` adapter is checked against the operation’s schemas: the arguments before the adapter runs, then its success value or error. A value a schema rejects is a defect, an `ExternalSystemSchemaError` that names the system, the operation and the side that broke:

```text
Payments.refund returned a value its success schema rejects
Expected string
  at ["refundId"]
```

A policy that sends bad arguments, or an adapter that maps a vendor response wrong, fails at the seam instead of deep inside the model. Fields the success schema doesn’t declare are dropped, so the model sees only what it declared. An error the operation doesn’t declare is a defect too. An operation without `success` returns the adapter’s value as it is.

Turn the check off for one adapter with `validate: false`, for example when the payload is large and the adapter already validates it:

```typescript
export const StripePayments = Payments.toLayer(build, { validate: false });
```

## Call it from a policy

A policy uses it like any other service. This one refunds money that settled on an account that had already closed.

```typescript
export const SettlementRefundLive = SettlementRefund.toLayer(
  Effect.gen(function* () {
    const payments = yield* Payments;
    return Effect.fn(function* ({ event }) {
      yield* payments.refund({ settlementId: event.settlementId, amount: event.amount });
    });
  }),
);
```

A policy can run more than once for the same event, so the call it makes can arrive twice. Pass the vendor something that identifies the request, here the `settlementId`, and have the adapter use it as the vendor’s idempotency key. However many times it’s asked, the money is refunded once. See [Make it safe to run twice](policy.md#make-it-safe-to-run-twice).

Provide the adapters where you host the model, next to the platform: see [Domain Model](model.md#run-it).

## Turn outside events into commands

Money arrives from Stripe as webhook events. Each successful payment becomes a `RecordSettlement`, in the model’s words. It reports a fact that already happened outside the model, so it declares no rejections.

```typescript
export class RecordSettlement extends Domain.Command("RecordSettlement", {
  target: AccountId,
  input: { settlementId: Schema.String, amount: Cents },
  events: [SettlementReceived, SettlementOnClosedAccount],
}) {}
```

The account can’t refuse money that has already settled, so it records what the settlement means for it. An open account records `SettlementReceived` with its new balance. A closed one records `SettlementOnClosedAccount`, and `SettlementRefund` sends the money back.

The webhook handler is the adapter for this direction. It receives each event, checks its signature, picks out what the model cares about, and sends the command:

```typescript
Effect.fn(function* (event) {
  const intent = event.object;
  const accountId = intent.metadata?.accountId;
  if (!accountId) return;
  yield* accounts.send(
    AccountId.make(accountId),
    new RecordSettlement({ settlementId: intent.id, amount: intent.amount_received }),
    { commandId: \`settlement:${intent.id}\` },
  );
}),
```

`Stripe.consumeEvents` receives the webhooks and checks their signatures before this handler runs. The payment’s id becomes the command id, so a webhook Stripe delivers twice settles once.

## Test external system calls

In a story, an external system call waits until you answer it. That puts every outcome of the outside world under your control: the refund that succeeds, the one that’s declined, and the one whose connection breaks.

```typescript
it.effect("money that settles on a closed account is refunded", () =>
  story(
    Story.given(
      account,
      new AccountOpened({ customerId: CustomerId.make("c-1"), owner: "sam" }),
      new AccountClosed(),
    ),
    Story.when(account, new RecordSettlement({ settlementId: "st_1", amount: 500 })),
    Story.then(account, new SettlementOnClosedAccount({ settlementId: "st_1", amount: 500 })),
    Story.call(Payments.refund, { settlementId: "st_1", amount: 500 }),
    Story.succeed(Payments.refund, { refundId: "re_1" }),
  ),
);
```

`Story.call` checks what was asked, and must come first; `succeed` answers it. `fail` answers with the operation’s declared error instead; see [Fail a call](testing-models.md#fail-a-call).

The story runner lists the systems it fakes in `stub`:

```typescript
const story = Story.make(Bank, {
  layer: BankHandlers.pipe(Layer.provide(MemoryLedgerStore)),
  stub: [FraudCheck, Payments],
});
```

A system you leave out of `stub` is called for real through the Layer you provide, and the story checks what each call was sent and returned; see [Assert real calls](testing-models.md#assert-real-calls). A story fails if any call is left unanswered or unasserted, so every outside call is accounted for.

Next, [Api](api.md): what the model publishes to its own clients.
