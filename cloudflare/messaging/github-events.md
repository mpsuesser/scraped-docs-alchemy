---
url: https://alchemy.run/cloudflare/messaging/github-events
title: "React to GitHub events from a Worker"
description: "Subscribe a Cloudflare Worker to GitHub repository webhooks with typed payloads — alchemy provisions the webhook and verifies delivery signatures."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A push lands on `main`, a pull request opens, a release tag appears — and you want code to run. `GitHub.consumeRepositoryEvents` subscribes a Cloudflare Worker to a repository’s webhooks: alchemy provisions the webhook on the repo, points it at the Worker, verifies each delivery’s signature, and hands your handler a fully typed payload.

This event source is **Cloudflare-only today** — see [the note below](#cloudflare-only-today).

<svg viewBox="0 0 508.79999999999995 230" style="max-width: 585px" role="img" aria-label="Alchemy creates the webhook with a secret only it and GitHub know. Deliveries GitHub signed reach your handler. Anything else gets a 401 before the handler runs."><defs><marker id="alc-bd-arrow-1u9zj9" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="220.39999999999998" y="118" width="272.4" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="158.8" y1="172" x2="234.39999999999998" y2="172" marker-end="url(#alc-bd-arrow-1u9zj9)"></line><text fill="currentColor" x="198.6" y="164" text-anchor="middle">signed</text></g> <g><path stroke="currentColor" fill="none" d="M 158.8 40 H 403.4 V 114"></path><path stroke="currentColor" fill="none" d="M 396.4 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="413.4" y="83">401</text></g> <text fill="currentColor" x="232.39999999999998" y="137" text-anchor="start">Deploys <tspan fill="currentColor" dx="8">Worker · checks the signature</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="87.4" y="37" text-anchor="middle">anyone else</text> <text fill="currentColor" x="87.4" y="53" text-anchor="middle">forged payload</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="87.4" y="169" text-anchor="middle">GitHub</text> <text fill="currentColor" x="87.4" y="185" text-anchor="middle">repository webhook</text></g> <g><rect stroke="currentColor" fill="none" x="238.39999999999998" y="148" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="309.79999999999995" y="169" text-anchor="middle">your handler</text> <text fill="currentColor" x="309.79999999999995" y="185" text-anchor="middle">typed push payload</text></g></svg> <svg viewBox="0 0 291.2 394" style="max-width: 335px" role="img" aria-label="Alchemy creates the webhook with a secret only it and GitHub know. Deliveries GitHub signed reach your handler. Anything else gets a 401 before the handler runs."><defs><marker id="alc-bd-arrow-hlvbbj" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="268" width="215.39999999999998" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="105.4" y1="212" x2="105.4" y2="308" marker-end="url(#alc-bd-arrow-hlvbbj)"></line><text fill="currentColor" x="115.4" y="266">signed</text></g> <g><path stroke="currentColor" fill="none" d="M 176.8 40 H 247.39999999999998 V 290 H 235.39999999999998"></path><path stroke="currentColor" fill="none" d="M 224.39999999999998 283 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="255.39999999999998" y="169">401</text></g> <text fill="currentColor" x="28" y="287" text-anchor="start">Deploys <tspan fill="currentColor" x="28" dy="14">Worker · checks the signature</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="105.4" y="37" text-anchor="middle">anyone else</text> <text fill="currentColor" x="105.4" y="53" text-anchor="middle">forged payload</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="105.4" y="185" text-anchor="middle">GitHub</text> <text fill="currentColor" x="105.4" y="201" text-anchor="middle">repository webhook</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="142.8" height="48" rx="8"></rect><text fill="currentColor" x="105.4" y="333" text-anchor="middle">your handler</text> <text fill="currentColor" x="105.4" y="349" text-anchor="middle">typed push payload</text></g></svg>

Alchemy creates the webhook with a secret only it and GitHub know. Deliveries GitHub signed reach your handler. Anything else gets a 401 before the handler runs.

Call `consumeRepositoryEvents` in the Worker’s Construction phase with the repository and the events you want, plus a handler that runs once per delivery:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as GitHub from "@alchemy.run/github";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    yield* GitHub.consumeRepositoryEvents(
      {
        owner: "my-org",
        repository: "my-repo",
        events: ["push"],
      },
      (event) => Effect.log(\`received ${event.name} (${event.id})\`),
    );

    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text("ok");
      }),
    };
  }),
);
```

`events` defaults to `["push"]`; pass `["*"]` to receive everything GitHub emits. The handler receives each delivery as a typed `WebhookEvent` and returns an `Effect` — more on the payload types below.

## 2\. Provide the Cloudflare runtime layer

`consumeRepositoryEvents` is host-agnostic — it requires a `RepositoryEventSource` implementation. Provide the Cloudflare one on the Worker:

```typescript
export default Cloudflare.Worker(
  "Worker",
  { main: import.meta.url },
  Effect.gen(function* () {
    // ...
  }),
  }).pipe(Effect.provide(GitHub.RepositoryEventSourceLive)),
);
```

`GitHub.RepositoryEventSourceLive` does both halves: at deploy time it provisions the repository webhook pointing at this Worker, and at runtime it registers a `fetch` listener that claims the webhook’s delivery path and forwards each verified delivery to your handler. Requests on any other path fall through to the Worker’s own `fetch`.

## 3\. Add GitHub providers to the Stack

The webhook is a GitHub resource, so the Stack needs GitHub providers alongside Cloudflare’s:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as GitHub from "@alchemy.run/github";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Worker from "./src/worker.ts";

export default Alchemy.Stack(
  "GitHubEvents",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), GitHub.providers()),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const worker = yield* Worker;
    return { url: worker.url.as<string>() };
  }),
);
```

`GitHub.providers()` resolves credentials from the environment, a stored PAT, or the `gh` CLI — see [GitHub setup](../../github/setup.md). The token needs `repo` scope (admin access to the repository) to manage webhooks.

## 4\. Verify deliveries with a secret

Without a secret, anyone who learns the delivery URL can forge events. Pass one and alchemy rejects any delivery whose `X-Hub-Signature-256` header doesn’t match:

```typescript
import * as Config from "effect/Config";

  Effect.gen(function* () {
    const secret = yield* Config.Redacted("GITHUB_WEBHOOK_SECRET");

    yield* GitHub.consumeRepositoryEvents(
      {
        owner: "my-org",
        repository: "my-repo",
        events: ["push"],
        secret,
      },
      (event) => Effect.log(\`received ${event.name} (${event.id})\`),
    );
```

The webhook is provisioned with the secret, so GitHub signs every delivery with `HMAC-SHA256`; the Worker recomputes the signature over the raw body and compares in constant time, answering `401` on a mismatch. `Config.Redacted` reads the value from your `.env` at deploy time and binds it as a Worker secret — see [Secrets & env](../security/secrets-env.md).

## 5\. Deploy

```sh
alchemy deploy
```

On deploy, alchemy creates a `GitHub.Webhook` on the repository whose delivery URL is the Worker’s URL plus a deterministic per-repo path (`/__alchemy/github/{owner}/{repository}` — override with the `path` prop). The webhook is updated in place on subsequent deploys and deleted when the stack is destroyed. Push a commit and watch the handler log the delivery.

## What alchemy automates

One `consumeRepositoryEvents` call replaces the usual webhook plumbing:

- **Provisioning** — a [`GitHub.Webhook`](https://alchemy.run/providers/github/reference/webhook#webhook) resource is created on the repo, wired to the Worker’s URL. You never paste a URL into repo settings.
- **Secret distribution** — the signing secret is bound onto the Worker as a secret under a deterministic env name; the runtime reads it back to verify signatures.
- **Routing** — the runtime only claims `POST` s on the delivery path (`405` for other methods, `202` once a delivery is accepted); every other request reaches your own `fetch` handler untouched.
- **Signature verification** — `HMAC-SHA256` over the raw body, compared in constant time against `X-Hub-Signature-256`.

## Typed payloads

`event` is a discriminated union of `{ id, name, payload }` keyed on `name`, narrowed to the events you selected. A `switch` on `event.name` narrows `payload` exhaustively:

```typescript
yield* GitHub.consumeRepositoryEvents(
  {
    owner: "my-org",
    repository: "my-repo",
    events: ["push", "pull_request"],
  },
  (event) => {
    switch (event.name) {
      case "push":
        return Effect.log(\`push to ${event.payload.ref}\`);
      case "pull_request":
        return Effect.log(\`PR: ${event.payload.pull_request.title}\`);
    }
  },
);
```

Selecting `["*"]` (or omitting `events`) widens the union back to every GitHub event name.

## Cloudflare-only today

`GitHub.RepositoryEventSourceLive` runs on any host that is a `WebhookHost`, and today only Cloudflare Workers are. On other hosts you can still provision a [`GitHub.Webhook`](https://alchemy.run/providers/github/reference/webhook#webhook) directly with any delivery URL, but signature verification and routing are yours to implement.

## Where next

- [GitHub provider](../../github.md) — repositories, Actions secrets and variables, and the rest of the CI/CD glue.
- [Workers](../compute/workers.md) — the host this event source runs on.
- [Webhook API reference](https://alchemy.run/providers/github/reference/webhook#webhook) — the resource provisioned under the hood.
