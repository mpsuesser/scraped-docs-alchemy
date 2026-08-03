---
url: https://alchemy.run/cloudflare
title: "Cloudflare"
description: "Build and deploy full applications on Cloudflare with Alchemy — one Worker runtime plus resources like Durable Objects, D1, R2, Queues, and Hyperdrive, wired together by typed bindings."
access_date: 2026-08-03T17:26:38.937Z
current_date: 2026-08-03T17:26:38.937Z
---

An alchemy app on Cloudflare is one **Worker** runtime plus the resources it talks to — databases, object storage, queues, stateful objects — all declared in the same TypeScript program and wired together by typed bindings. Deploy the whole thing with `bun alchemy deploy`; Alchemy figures out what changed.

New here? [Set up your account](https://alchemy.run/cloudflare/setup), then start the [tutorial](https://alchemy.run/cloudflare/tutorial/part-1).

## Compute

- [Workers](https://alchemy.run/cloudflare/compute/workers) — the compute runtime; every app has at least one.
- [Durable Objects](https://alchemy.run/cloudflare/compute/durable-objects) — globally-unique stateful instances with transactional storage and typed RPC.
- [Containers](https://alchemy.run/cloudflare/compute/containers) — long-lived processes and arbitrary runtimes, paired with a Durable Object for typed RPC.
- [Workflows](https://alchemy.run/cloudflare/compute/workflows) — durable multi-step jobs with checkpointed, replayable steps.

## Frontend

- [Vite](https://alchemy.run/cloudflare/frontend/vite) — deploy any pure-Vite app (SPAs, TanStack Start, React Router, SolidStart) as a Worker with assets.
- [Static sites](https://alchemy.run/cloudflare/frontend/static-site) — any build command’s static output, including frameworks Vite doesn’t cover yet.

## Data

- [D1](https://alchemy.run/cloudflare/data/d1) — serverless SQLite with migrations managed as part of the deploy.
- [KV](https://alchemy.run/cloudflare/data/kv) — edge key-value storage for config, sessions, and cached lookups.
- [R2](https://alchemy.run/cloudflare/data/r2) — object storage with read/write-scoped bindings.
- [Hyperdrive](https://alchemy.run/cloudflare/data/hyperdrive) — edge connection pooling for external Postgres and MySQL.

## Messaging

- [Queues](https://alchemy.run/cloudflare/messaging/queues) — at-least-once message delivery between Workers.

## Networking

- [Domains & DNS](https://alchemy.run/cloudflare/networking/domains) — zones, DNS records, and zone settings as resources; adopt the domains you already own.

## Email

- [Email](https://alchemy.run/cloudflare/email) — route inbound mail on a zone you own and send from Workers.

## Security & secrets

- [Secrets & env](https://alchemy.run/cloudflare/security/secrets-env) — bind `.env` values and secrets into your Workers.

## What are you building?

| App shape | Stack |
| --- | --- |
| HTTP API | [Worker](https://alchemy.run/cloudflare/compute/workers) + [D1](https://alchemy.run/cloudflare/data/d1) — see [Effect HTTP API](https://alchemy.run/cloudflare/apis/effect-http-api) |
| Call one Worker from another (internal RPC) | [Worker](https://alchemy.run/cloudflare/compute/workers) — see [Schemaless RPC](https://alchemy.run/cloudflare/compute/workers#schemaless-rpc) and the [concept](https://alchemy.run/apis/schemaless) |
| Typed API for external clients | [Effect RPC](https://alchemy.run/cloudflare/apis/effect-rpc) for Effect clients, [Effect HTTP API](https://alchemy.run/cloudflare/apis/effect-http-api) for plain HTTP |
| Real-time / WebSockets | [Durable Objects](https://alchemy.run/cloudflare/compute/durable-objects) — see [Accept WebSockets](https://alchemy.run/cloudflare/compute/hibernatable-websockets) |
| Full-stack app | [Worker](https://alchemy.run/cloudflare/compute/workers) + a React SPA — see [Add a React SPA](https://alchemy.run/cloudflare/frontend/vite-spa) or [Frontend frameworks](https://alchemy.run/cloudflare/frontend/frontends) |
| Background jobs | [Queues](https://alchemy.run/cloudflare/messaging/queues) |
| Postgres-backed API | [Hyperdrive](https://alchemy.run/cloudflare/data/hyperdrive) + [Drizzle](https://alchemy.run/cloudflare/data/drizzle) |
| Durable multi-step jobs | [Workflows](https://alchemy.run/cloudflare/compute/workflows) |
| AI apps | [AI Gateway](https://alchemy.run/cloudflare/ai/ai-gateway), [Effect AI](https://alchemy.run/cloudflare/ai/effect-ai), [AI Search](https://alchemy.run/cloudflare/ai/ai-search) |
| App on your own domain | [Domains & DNS](https://alchemy.run/cloudflare/networking/domains) — see [Custom domains & routes](https://alchemy.run/cloudflare/networking/custom-domains) |
| Production observability | [Ship Worker telemetry to Axiom](https://alchemy.run/cloudflare/observability/axiom-observability) |

For frontends, alchemy is Workers-first: Pages resources exist, but static assets served from a [Worker](https://alchemy.run/cloudflare/compute/workers) is the recommended path.

## Where next

- [Setup](https://alchemy.run/cloudflare/setup) — install alchemy and connect your Cloudflare account.
- [Tutorial](https://alchemy.run/cloudflare/tutorial/part-1) — from empty directory to a deployed Worker with tests, stages, and CI.
- [Custom domains & routes](https://alchemy.run/cloudflare/networking/custom-domains) — put your Worker on a domain you own.
- [Ship Worker telemetry to Axiom](https://alchemy.run/cloudflare/observability/axiom-observability) — datasets, ingest tokens, and monitors in the same Stack as the Worker.
- [API reference](https://alchemy.run/providers) — every Cloudflare resource and binding.

## Everything else

The blocks above are what most apps are built from, but coverage goes much further: the zero-trust and zone-security long tail — roughly 60 more services, produced by the resource factory — ships as reference-only documentation in the [API reference](https://alchemy.run/providers).
