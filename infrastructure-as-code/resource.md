---
url: https://alchemy.run/infrastructure-as-code/resource
title: "Resources"
description: "Resources are named cloud entities with input properties and output attributes."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

A **Resource** represents a cloud entity managed by Alchemy — a bucket, database, queue, function, DNS record, or anything else that has a lifecycle of reconcile and delete.

## Declaring a Resource

Resources are declared with a **logical ID** and optional **input properties**:

```typescript
const bucket = yield* Cloudflare.R2.Bucket("Bucket");
const queue = yield* AWS.SQS.Queue("Jobs", {
  fifoQueue: true,
});
```

The logical ID (`"Bucket"`, `"Jobs"`) is stable across deploys. It identifies this resource within the stack and is used to track state.

## Input properties and output attributes

Every resource has two sides:

- **Input Properties** — the desired configuration you pass in (e.g. `fifoQueue: true`)
- **Output Attributes** — the values produced after creation (e.g. `queueUrl`, `queueArn`)

Output attributes are available as [`Output`](outputs.md) expressions on the resource — lazy, typed references that resolve once the upstream resource has been created:

```typescript
const bucket = yield* Cloudflare.R2.Bucket("Bucket");
bucket.bucketName; // Output<string>
```

See [Inputs & Outputs](outputs.md) for the full set of operators (`map`, `mapEffect`, `all`, `interpolate`, `ref`).

These are lazy references that resolve after the resource is created. You can pass them as inputs to other resources to express dependencies.

## Resources are Effects

A resource declaration like `Cloudflare.R2.Bucket("Bucket")` is just an `Effect` — calling it doesn’t talk to the cloud. `yield*` -ing it inside a [Stack](stack.md) doesn’t either; it just **registers the resource on the stack** and hands you back a typed [`Output`](outputs.md) reference for its attributes:

```typescript
// 1. Build the Effect. No API calls. No state mutation.
const Bucket = Cloudflare.R2.Bucket("Bucket");

// 2. Register it on the stack. Still no API calls — alchemy is
//    just collecting the desired-state graph.
export default Alchemy.Stack(
  "MyApp",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    const bucket = yield* Bucket;
    return { name: bucket.bucketName };
  }),
);
```

The cloud is only touched later, when `alchemy deploy` runs the collected graph through plan and apply. See [Resource Lifecycle](resource-lifecycle.md) for what happens after registration.

### Sharing across files

Because the declaration is just a value, you can `export` it and import it from anywhere — handlers, layers, other resources:

```typescript
export const Bucket = Cloudflare.R2.Bucket("Bucket");
```

```typescript
import { Bucket } from "./src/bucket.ts";

export default Alchemy.Stack(
  "MyApp",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    yield* Bucket;
  }),
);
```

Importing the same `Bucket` from multiple files is safe. Alchemy keys resources by their fully qualified name, so even if two modules `yield*` it, it registers on the stack exactly once.

## ref

Every resource constructor exposes a static `ref` returning a typed reference to a resource that’s already been deployed **elsewhere** — another stage of this stack, or a different stack entirely:

```typescript
const project = yield* Neon.Project.ref("app-db", {
  stage: "staging",
});
// project: Neon.Project — same shape, same typed attributes
```

Lookup keyed by `{ stack, stage, id }`; both options default to the current stack/stage. Lazy — no cloud calls at declaration time. Typed — the result has the same interface as a freshly deployed resource. Strict — fails fast with `InvalidReferenceError` when the target is missing.

A typical pattern: PR-preview stages reference long-lived `staging` resources instead of provisioning their own:

```typescript
const project = stage.startsWith("pr-")
  ? yield* Neon.Project.ref("app-db", { stage: "staging" })
  : yield* Neon.Project("app-db", { region: "aws-us-east-1" });
```

See [References](references.md) for the full reference surface and the [Shared database across stages](../cloudflare/data/shared-database.md) guide for the canonical walkthrough.

## Logical ID

The first argument you pass to a resource constructor is its **logical ID** — a name **you** choose to identify the resource within its stack:

```typescript
const Bucket = Cloudflare.R2.Bucket("Bucket"); // logical ID: "Bucket"
const Jobs = AWS.SQS.Queue("Jobs");           // logical ID: "Jobs"
```

The logical ID is how alchemy tracks the resource in state across deploys:

- **Stable across deploys** — keep the same ID and alchemy keeps updating the same underlying cloud resource.
- **Stable across renames** — change the variable name, change the TypeScript class, move the file; as long as the logical ID stays the same, alchemy still recognizes it.
- **Rename = replace** — change the logical ID and alchemy treats it as a new resource (and deletes the old one on the next deploy).

Logical IDs only need to be unique **within a stack**.

## Physical name

The **physical name** is what the cloud actually sees — `myapp-dev_sam-bucket-a3f1` on R2, an ARN suffix on AWS, etc. Alchemy generates it for you from three things:

```text
{stack-name}-{stage}-{logical-id}-{instance-id}
   "myapp"    "dev_sam"  "Bucket"     "a3f1"
```

The first three are obvious. The **instance ID** is a short, deterministic suffix tied to *this specific instance* of the resource. While the resource lives, the instance ID stays the same, so re-running create finds the existing resource instead of duplicating it.

The whole scheme means:

- **Stages don’t collide** — `dev_sam` and `prod` produce different physical names from the same code.
- **Creates are idempotent** — same logical ID + same instance ID = same physical name on retry.
- **State can recover** — if persistence fails, alchemy can re-run create and find the existing cloud resource.

The instance ID is the part that **does** change when a resource is replaced — which leads us to…

## Replacement

Some property changes can’t be applied in place. Changing a DynamoDB table’s partition key, for example, can’t be done on a live table — it has to be re-created.

Before:

```typescript
const Jobs = DynamoDB.Table("Jobs", {
  partitionKey: "id",
  attributes: { id: "S" },
});
```

After:

```typescript
const Jobs = DynamoDB.Table("Jobs", {
  partitionKey: "id",
  attributes: { id: "S" },
  partitionKey: "tenantId",
  attributes: { tenantId: "S" },
});
```

The logical ID (`"Jobs"`) doesn’t change, but the **instance ID does** — which means the physical name does too:

```text
before:  myapp-prod-jobs-a3f1
after:   myapp-prod-jobs-9b2c
```

When the next plan runs, alchemy:

1. Creates a new table with the new instance ID (and physical name)
2. Updates downstream resources to reference the new one
3. Deletes the old table

The resource’s [provider](provider.md) decides which property changes trigger replacement vs in-place update (via [`diff`](provider.md#diff)). For the full lifecycle (reconcile / replace / delete) see [Resource Lifecycle](resource-lifecycle.md).

## Defining your own Resource type

A resource is just a typed Effect. To support a new cloud or third-party API, declare a `Resource` type with its input props and output attributes — then implement its provider as a `Layer`. Same engine plans, deploys, and destroys it.

See [Writing a Custom Resource Provider](custom-provider.md) for a step-by-step walkthrough of declaring the type and implementing each lifecycle hook (`reconcile`, `delete`, `diff`, `read`).

```typescript
// 1. Declare the type + constructor.
export type StripeProduct = Resource<
  "Stripe.Product",
  { name: string; price: number }, // input props
  { productId: string; priceId: string } // output attrs
>;
export const StripeProduct = Resource<StripeProduct>("Stripe.Product");

// 2. Use it like any built-in resource.
const Pro = yield* StripeProduct("Pro", {
  name: "Pro plan",
  price: 2900,
});
// ^? typed Pro.productId, Pro.priceId
```

- **Inputs & outputs are typed** — Props you pass in, attributes the provider returns. Both fully typed, both checked at the call site.
- **Compose with built-in providers** — Merge your provider Layer with `Cloudflare.providers()` or `AWS.providers()`. One stack, mixed clouds.

The lifecycle hooks the provider implements — `reconcile`, `delete`, `diff`, `read` — are documented in [Provider](provider.md#lifecycle-operations).

## The resource graph

Every attribute a resource exposes (`Bucket.bucketName`, `Worker.url`) is an [Output](outputs.md) — a lazy, typed reference that resolves at deploy time. Passing an Output from one resource as input to another draws an edge in the dependency graph. Take this stack:

```typescript
const Bucket = yield* Cloudflare.R2.Bucket("Bucket");
const Sessions = yield* Cloudflare.KV.Namespace("Sessions");

const Queue = yield* AWS.SQS.Queue("Queue", {
  name: Output.interpolate\`${Bucket.bucketName}-events\`,
});

const Worker = yield* Cloudflare.Worker("Worker", {
  main: import.meta.url,
  env: { Bucket, Sessions, Queue },
});
```

Alchemy reads the Outputs in each resource’s props and builds:

<svg viewBox="0 0 628 204" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 C 201 56, 201 102, 238 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 164 56 C 311 56, 311 102, 458 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 164 148 C 311 148, 311 102, 458 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g><path d="M 384 102 C 421 102, 421 102, 458 102" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path></g><g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Bucket</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloudflare.R2.Bucket</text></g> <g transform="translate(24, 116)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Sessions</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloudflare.KV.Namespace</text></g> <g transform="translate(244, 70)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Queue</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">AWS.SQS.Queue</text></g> <g transform="translate(464, 70)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloudflare.Worker</text></g></svg>

It then deploys in topological order:

1. `Bucket` and `Sessions` have no dependencies → created **in parallel**.
2. `Queue` depends on `Bucket.bucketName` → waits for `Bucket`, then created.
3. `Worker` depends on all three → created last, after every upstream Output has resolved.

Outputs draw the edges. [Bindings](../infrastructure-as-effects/binding.md) attach resources to Workers and Lambdas — and when bindings form a cycle, alchemy handles it with a two-phase plan; see [Circular Bindings](../infrastructure-as-effects/circular-bindings.md).

## Circular references

Real systems have cycles — two Workers that call each other, a Lambda that invokes another Lambda. Alchemy resolves them by splitting each Function resource ([Functions & Servers](../infrastructure-as-effects/functions-and-servers.md)) into a **class** that acts as the Tag (the identity) and a **`.make(...)`** Layer that supplies the runtime implementation — so the class can be referenced before its implementation exists:

<svg viewBox="0 0 464 168" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 314 88 C 314 130, 94 130, 94 94" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="204" y="124" text-anchor="middle">binds</text></g> <g><path d="M 164 56 C 201 56, 201 56, 238 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow)"></path><text fill="currentColor" x="201" y="50" text-anchor="middle">binds</text></g> <g transform="translate(244, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker A</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloudflare.Worker</text></g> <g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker B</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloudflare.Worker</text></g></svg>

See [Circular Bindings](../infrastructure-as-effects/circular-bindings.md) for the full A↔B build-up and how alchemy plans the two-phase create-then-wire deploy.

## Where next

- [Actions](action.md) — deploy-time work without a lifecycle.
- [Inputs & Outputs](outputs.md) — the lazy references that draw the graph.
- [Resource lifecycle](resource-lifecycle.md) — what happens on deploy.
