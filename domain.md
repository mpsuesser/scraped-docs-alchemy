---
url: https://alchemy.run/domain
title: "Domain"
description: "A domain-driven design framework for Effect. Model your business as commands, aggregates, events, views, projections and policies, test it in memory, and run it on Cloudflare Durable Objects."
access_date: 2026-11-02T21:06:06.110Z
current_date: 2026-11-02T21:06:06.110Z
---

`@alchemy.run/domain` models your business in its own terms. Commands ask for changes, aggregates decide them, and events record what happened. Views, projections and policies then answer questions, feed your own stores, and apply the business’s rules about what to do next.

The same model runs in memory for tests and on Cloudflare Durable Objects in production, with one ordered partition per key.

<svg viewBox="0 0 700 232" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 188 56 L 262 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 432 56 L 506 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 594 88 L 594 176 L 438 176" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 268 176 L 194 176" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><g style="--hue:#4f9fd8"><path fill="none" d="M 6 0 H 152 L 164 32 L 152 64 H 6 Q 0 64 0 58 V 6 Q 0 0 6 0 Z"></path><g transform="translate(40.8, 21) scale(0.5833333333333334)"><path stroke="currentColor" fill="none" d="M21 3L10 14"></path><path fill="none" d="M21 3l-6.5 18-4.5-7-7-4.5z"></path></g><text fill="currentColor" x="60.8" y="28" dominant-baseline="middle">Commands</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">change something</text></g></g> <g transform="translate(268, 24)"><g style="--hue:#e3b341"><rect stroke="currentColor" fill="none" width="164" height="64" rx="8"></rect><rect stroke="currentColor" fill="none" x="4" y="4" width="156" height="56" rx="5"></rect><g transform="translate(33, 21) scale(0.5833333333333334)"><path fill="none" d="M12 3l8 4.5v9L12 21l-8-4.5v-9z"></path><path stroke="currentColor" fill="none" d="M4 7.5l8 4.5 8-4.5"></path><path stroke="currentColor" fill="none" d="M12 12v9"></path></g><text fill="currentColor" x="53" y="28" dominant-baseline="middle">Aggregates</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">decide</text></g></g> <g transform="translate(512, 24)"><g style="--hue:#f0782f"><path fill="none" d="M 4 0 H 151 L 164 13 V 60 Q 164 64 160 64 H 4 Q 0 64 0 60 V 4 Q 0 0 4 0 Z"></path><path stroke="currentColor" fill="none" d="M 151 0 V 13 H 164"></path><g transform="translate(48.6, 21) scale(0.5833333333333334)"><path fill="none" d="M13 2L4 14h7l-1 8 9-12h-7z"></path></g><text fill="currentColor" x="68.6" y="28" dominant-baseline="middle">Events</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">the record</text></g></g> <g transform="translate(268, 144)"><g style="--hue:#7fb069"><rect stroke="currentColor" fill="none" width="164" height="64" rx="32"></rect><g transform="translate(52.5, 21) scale(0.5833333333333334)"><path fill="none" d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"></path><path fill="none" d="M12 9a3 3 0 1 0 0 6a3 3 0 1 0 0-6z"></path></g><text fill="currentColor" x="72.5" y="28" dominant-baseline="middle">Views</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">answer reads</text></g></g> <g transform="translate(24, 144)"><g style="--hue:#e6d3a8"><path fill="none" d="M 12 0 H 158 Q 164 0 164 6 V 58 Q 164 64 158 64 H 12 L 0 32 Z"></path><g transform="translate(44.7, 21) scale(0.5833333333333334)"><path fill="none" d="M11 4a7 7 0 1 0 0 14a7 7 0 1 0 0-14z"></path><path stroke="currentColor" fill="none" d="M20 20l-4-4"></path></g><text fill="currentColor" x="64.7" y="28" dominant-baseline="middle">Queries</text> <text fill="currentColor" x="82" y="46" text-anchor="middle" dominant-baseline="middle">read the views</text></g></g></svg>

CommandAggregateEventViewQuery

## A taste

A command declares what it asks for and every way it can end:

```typescript
export class MoneyDeposited extends Domain.Event("MoneyDeposited", {
  amount: Schema.Int,
  balanceAfter: Schema.Int,
}) {}

export class DepositMoney extends Domain.Command("DepositMoney", {
  target: AccountId,
  input: { amount: Schema.Int },
  events: [MoneyDeposited],
  rejects: [NotOpen],
  reply: { balance: Schema.Int },
}) {}
```

The account aggregate decides it against its current state:

```typescript
DepositMoney: (s, cmd) =>
  s._tag !== "Open"
    ? new NotOpen()
    : [new MoneyDeposited({ amount: cmd.amount, balanceAfter: s.balance + cmd.amount })],
```

And a story tests it in memory, in the business’s own words: given this history, when this command, then these events.

```typescript
it.effect("deposits into an open account and replies with the balance", () =>
  story(
    Story.given(account, new AccountOpened({ owner: "sam" })),
    Story.when(account, new DepositMoney({ amount: 100 })),
    Story.then(account, new MoneyDeposited({ amount: 100, balanceAfter: 100 })),
    Story.replied({ balance: 100 }),
  ),
);
```

## Build it with your coding agent

The `alchemy-domain` skill teaches a coding agent how to turn requirements into a model, lay out the project, test it with stories, host it on Cloudflare and drive it from foldkit.

Ship it with your dependencies, so the skill always matches the alchemy version you have installed:

```sh
bun add -d skills-npm
bunx skills-npm setup
```

`setup` adds a `prepare` script. Every install then links the skills your packages ship into each coding agent’s skills folder.

Or install it from GitHub, for any agent:

```sh
npx skills add alchemy-run/alchemy --skill alchemy-domain
```

In Claude Code, it’s also a plugin:

```text
/plugin install alchemy --marketplace alchemy-run/alchemy
```

A skill installed from GitHub follows the repository’s main branch, so update it when you upgrade alchemy.

## Where to start

**New to domain-driven design?** [What is Domain-Driven Design?](domain/what-is-ddd.md) introduces the ideas one at a time, with the bank as the example.

**Want to build one?** The [tutorial](domain/tutorial/part-1.md) builds a bank step by step, from a single account on your machine to an Api deployed on Cloudflare with a browser app.

**Want the whole picture?** The concepts read as a manual, one building block per chapter, starting with [Domain Model](domain/model.md) and ending with [Story](domain/story.md). Each chapter explains the idea, how to find it in your business, and how to write it: [Command](domain/command.md), [Event](domain/event.md), [Aggregate](domain/aggregate.md), [View](domain/view.md), [Projection](domain/projection.md), [Stream](domain/stream.md), [Policy](domain/policy.md), [Saga](domain/saga.md), [External System](domain/external-system.md) and [Api](domain/api.md).
