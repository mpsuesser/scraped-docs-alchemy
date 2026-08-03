---
url: https://alchemy.run/cloudflare
title: "Cloudflare"
description: "Build and deploy full applications on Cloudflare with Alchemy — one Worker runtime plus resources like Durable Objects, D1, R2, Queues, and Hyperdrive, wired together by typed bindings."
access_date: 2026-08-03T18:54:18.847Z
current_date: 2026-08-03T18:54:18.847Z
---

An alchemy app on Cloudflare is one **Worker** runtime plus the resources it talks to — databases, object storage, queues, stateful objects — all declared in the same TypeScript program and wired together by typed bindings. Deploy the whole thing with `bun alchemy deploy`; Alchemy figures out what changed.

New here? [Set up your account](cloudflare/setup.md), then start the [tutorial](cloudflare/tutorial/part-1.md).

## Compute

- [Workers](cloudflare/compute/workers.md) — the compute runtime; every app has at least one.
- [Durable Objects](cloudflare/compute/durable-objects.md) — globally-unique stateful instances with transactional storage and typed RPC.
- [Containers](cloudflare/compute/containers.md) — long-lived processes and arbitrary runtimes, paired with a Durable Object for typed RPC.
- [Workflows](cloudflare/compute/workflows.md) — durable multi-step jobs with checkpointed, replayable steps.

## Frontend

- [Vite](cloudflare/frontend/vite.md) — deploy any pure-Vite app (SPAs, TanStack Start, React Router, SolidStart) as a Worker with assets.
- [Static sites](cloudflare/frontend/static-site.md) — any build command’s static output, including frameworks Vite doesn’t cover yet.

## Data

- [D1](cloudflare/data/d1.md) — serverless SQLite with migrations managed as part of the deploy.
- [KV](cloudflare/data/kv.md) — edge key-value storage for config, sessions, and cached lookups.
- [R2](cloudflare/data/r2.md) — object storage with read/write-scoped bindings.
- [Hyperdrive](cloudflare/data/hyperdrive.md) — edge connection pooling for external Postgres and MySQL.

## Messaging

- [Queues](cloudflare/messaging/queues.md) — at-least-once message delivery between Workers.

## Networking

- [Domains & DNS](cloudflare/networking/domains.md) — zones, DNS records, and zone settings as resources; adopt the domains you already own.

## Email

- [Email](cloudflare/email.md) — route inbound mail on a zone you own and send from Workers.

## Security & secrets

- [Secrets & env](cloudflare/security/secrets-env.md) — bind `.env` values and secrets into your Workers.

## What are you building?

| App shape | Stack |
| --- | --- |
| HTTP API | [Worker](cloudflare/compute/workers.md) + [D1](cloudflare/data/d1.md) — see [Effect HTTP API](cloudflare/apis/effect-http-api.md) |
| Call one Worker from another (internal RPC) | [Worker](cloudflare/compute/workers.md) — see [Schemaless RPC](cloudflare/compute/workers.md#schemaless-rpc) and the [concept](apis/schemaless.md) |
| Typed API for external clients | [Effect RPC](cloudflare/apis/effect-rpc.md) for Effect clients, [Effect HTTP API](cloudflare/apis/effect-http-api.md) for plain HTTP |
| Real-time / WebSockets | [Durable Objects](cloudflare/compute/durable-objects.md) — see [Accept WebSockets](cloudflare/compute/hibernatable-websockets.md) |
| Full-stack app | [Worker](cloudflare/compute/workers.md) + a React SPA — see [Add a React SPA](cloudflare/frontend/vite-spa.md) or [Frontend frameworks](cloudflare/frontend/frontends.md) |
| Background jobs | [Queues](cloudflare/messaging/queues.md) |
| Postgres-backed API | [Hyperdrive](cloudflare/data/hyperdrive.md) + [Drizzle](cloudflare/data/drizzle.md) |
| Durable multi-step jobs | [Workflows](cloudflare/compute/workflows.md) |
| AI apps | [AI Gateway](cloudflare/ai/ai-gateway.md), [Effect AI](cloudflare/ai/effect-ai.md), [AI Search](cloudflare/ai/ai-search.md) |
| App on your own domain | [Domains & DNS](cloudflare/networking/domains.md) — see [Custom domains & routes](cloudflare/networking/custom-domains.md) |
| Production observability | [Ship Worker telemetry to Axiom](cloudflare/observability/axiom-observability.md) |

For frontends, alchemy is Workers-first: Pages resources exist, but static assets served from a [Worker](cloudflare/compute/workers.md) is the recommended path.

## Where next

- [Setup](cloudflare/setup.md) — install alchemy and connect your Cloudflare account.
- [Tutorial](cloudflare/tutorial/part-1.md) — from empty directory to a deployed Worker with tests, stages, and CI.
- [Custom domains & routes](cloudflare/networking/custom-domains.md) — put your Worker on a domain you own.
- [Ship Worker telemetry to Axiom](cloudflare/observability/axiom-observability.md) — datasets, ingest tokens, and monitors in the same Stack as the Worker.
- [API reference](https://alchemy.run/providers) — every Cloudflare resource and binding.

## Everything else

The blocks above are what most apps are built from, but coverage goes much further: the zero-trust and zone-security long tail — roughly 60 more services, produced by the resource factory — ships as reference-only documentation in the [API reference](https://alchemy.run/providers).
