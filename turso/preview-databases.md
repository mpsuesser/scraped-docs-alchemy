---
url: https://alchemy.run/turso/preview-databases
title: "Preview databases"
description: "Give every pull request its own copy of your staging database."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Give every pull request a copy of your staging database. Each preview gets real data to test with, and is deleted when you’re done.

<svg viewBox="0 0 633.2 192" style="max-width: 728px" role="img" aria-label="Each preview copies staging's database as it is when the PR deploys, runs the PR's own migrations on the copy, and is deleted with the PR. Staging never changes."><g><text fill="currentColor" x="4" y="30">staging</text> <text fill="currentColor" x="4" y="44">database</text> <line stroke-opacity="0.2" stroke="currentColor" x1="77.19999999999999" y1="30" x2="617.2" y2="30" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="4" y="94">pr-12</text> <text fill="currentColor" x="4" y="108">copy</text> <path stroke="currentColor" fill="none" d="M 212.2 30 C 230.2 30, 220.2 94, 238.2 94"></path><line stroke-opacity="0.2" stroke="currentColor" x1="238.2" y1="94" x2="428.2" y2="94"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="423.2" y1="89" x2="433.2" y2="99"></line><line stroke-opacity="0.2" stroke="currentColor" x1="423.2" y1="99" x2="433.2" y2="89"></line></g></g><g><text fill="currentColor" x="4" y="158">pr-15</text> <text fill="currentColor" x="4" y="172">copy</text> <path stroke="currentColor" fill="none" d="M 347.2 30 C 365.2 30, 355.2 158, 373.2 158"></path><line stroke-opacity="0.2" stroke="currentColor" x1="373.2" y1="158" x2="617.2" y2="158"></line></g><g><circle stroke="currentColor" fill="none" cx="314.8" cy="94" r="5.5"></circle><text fill="currentColor" x="314.8" y="83" text-anchor="middle">PR's migrations</text></g> <g><circle stroke="currentColor" fill="none" cx="428.2" cy="94" r="5.5"></circle><text fill="currentColor" x="428.2" y="115" text-anchor="middle">PR closed</text></g> <g><circle stroke="currentColor" fill="none" cx="487.59999999999997" cy="158" r="5.5"></circle><text fill="currentColor" x="487.59999999999997" y="147" text-anchor="middle">PR's migrations</text></g></svg> <svg viewBox="0 0 328 236" style="max-width: 377px" role="img" aria-label="Each preview copies staging's database as it is when the PR deploys, runs the PR's own migrations on the copy, and is deleted with the PR. Staging never changes."><g><text fill="currentColor" x="0" y="16">staging <tspan fill="currentColor" dx="8">database</tspan></text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="46" x2="312" y2="46" style="stroke-width: 3"></line></g><g><text fill="currentColor" x="110" y="94">pr-12 <tspan fill="currentColor" dx="8">copy</tspan></text> <path stroke="currentColor" fill="none" d="M 84 46 C 102 46, 92 124, 110 124"></path><line stroke-opacity="0.2" stroke="currentColor" x1="110" y1="124" x2="205.6" y2="124"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="200.6" y1="119" x2="210.6" y2="129"></line><line stroke-opacity="0.2" stroke="currentColor" x1="200.6" y1="129" x2="210.6" y2="119"></line></g></g><g><text fill="currentColor" x="186" y="172">pr-15 <tspan fill="currentColor" dx="8">copy</tspan></text> <path stroke="currentColor" fill="none" d="M 160 46 C 178 46, 168 202, 186 202"></path><line stroke-opacity="0.2" stroke="currentColor" x1="186" y1="202" x2="312" y2="202"></line></g><g><circle stroke="currentColor" fill="none" cx="141.76000000000002" cy="124" r="5.5"></circle><text fill="currentColor" x="93.76000000000002" y="113" text-anchor="start">PR's migrations</text></g> <g><circle stroke="currentColor" fill="none" cx="205.6" cy="124" r="5.5"></circle><text fill="currentColor" x="176.79999999999998" y="145" text-anchor="start">PR closed</text></g> <g><circle stroke="currentColor" fill="none" cx="239.04" cy="202" r="5.5"></circle><text fill="currentColor" x="191.04" y="191" text-anchor="start">PR's migrations</text></g></svg>

Each preview copies staging's database as it is when the PR deploys, runs the PR's own migrations on the copy, and is deleted with the PR. Staging never changes.

## Read the stage

```typescript
import * as Alchemy from "alchemy";

const { stage } = yield* Alchemy.Stack;
const preview = stage.startsWith("pr-");
```

## Reuse staging’s group

```typescript
const group = preview
  ? yield* Turso.Group.ref("Group", { stage: "staging" })
  : yield* Turso.Group("Group", { location: "aws-us-east-1" });
```

`Turso.Group.ref` uses the group from the `staging` stage. Previews never create or delete it.

## Copy staging’s database

```typescript
const source = preview ? yield* Turso.Database.ref("Db", { stage: "staging" }) : undefined;

const db = yield* Turso.Database("Db", {
  group: group.name,
  seed: source ? { type: "database", name: source.name } : undefined,
  migrations: "./migrations",
});
```

Each preview gets its own copy of staging’s database, then runs any new migrations from the pull request.

## Tear it down

```sh
alchemy destroy --stage pr-42
```

This deletes the preview’s database. Staging’s group and database stay.

## Where next

- [Databases](databases.md#copy-another-database)
- [Migrations](migrations.md)
