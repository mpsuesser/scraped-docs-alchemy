---
url: https://alchemy.run/prisma/compute/deployments
title: "Deployments"
description: "The fail-closed Compute deployment lifecycle — health-checked promotion, rollback recovery, old-generation cleanup, and env-only redeploys."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

When the artifact or deployment configuration changes, `Prisma.Compute` creates a new generation and walks it through a fail-closed sequence: upload the artifact, start the candidate, verify its preview health endpoint, promote it, confirm the app reports it as latest, verify the stable endpoint — and only then touch the previous deployment.

<svg viewBox="0 0 704.8 124" style="max-width: 811px" role="img" aria-label="The previous deployment keeps serving until the candidate is healthy, promoted, and verified on the stable endpoint. Only then is the old one touched."><line stroke-opacity="0.2" stroke="currentColor" x1="412.8" y1="24" x2="412.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="558.4000000000001" y1="24" x2="558.4000000000001" y2="118"></line><g><text fill="currentColor" x="4" y="50">previous</text> <text fill="currentColor" x="4" y="64">stable deployment</text> <g><rect stroke="currentColor" fill="none" x="132.8" y="40" width="425.6000000000001" height="24" rx="5"></rect><text fill="currentColor" x="345.6" y="56" text-anchor="middle">serving the stable endpoint</text></g> <g><rect stroke="currentColor" fill="none" x="558.4000000000001" y="40" width="78.39999999999986" height="24" rx="5"></rect><text fill="currentColor" x="597.6" y="56" text-anchor="middle">removed</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="704.8" y2="74"></line><text fill="currentColor" x="4" y="94">candidate</text> <text fill="currentColor" x="4" y="108">new generation</text> <g><rect stroke="currentColor" fill="none" x="166.4" y="84" width="78.4" height="24" rx="5"></rect><text fill="currentColor" x="205.60000000000002" y="100" text-anchor="middle">upload</text></g> <g><rect stroke="currentColor" fill="none" x="244.8" y="84" width="67.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="278.4" y="100" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="312" y="84" width="100.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="362.4" y="100" text-anchor="middle">health</text></g> <g><rect stroke="currentColor" fill="none" x="412.8" y="84" width="279.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="552.8" y="100" text-anchor="middle">promoted · serving</text></g></g> <text fill="currentColor" x="412.8" y="18" text-anchor="middle">promote</text> <text fill="currentColor" x="558.4000000000001" y="18" text-anchor="middle">stable endpoint verified</text></svg> <svg viewBox="0 0 330 188" style="max-width: 379px" role="img" aria-label="The previous deployment keeps serving until the candidate is healthy, promoted, and verified on the stable endpoint. Only then is the old one touched."><line stroke-opacity="0.2" stroke="currentColor" x1="159" y1="38" x2="159" y2="182"></line><line stroke-opacity="0.2" stroke="currentColor" x1="241.68" y1="38" x2="241.68" y2="182"></line><g><text fill="currentColor" x="0" y="58">previous <tspan fill="currentColor" dx="8">stable deployment</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="68" width="241.68" height="24" rx="5"></rect><text fill="currentColor" x="120.84" y="84" text-anchor="middle">serving the stable endpoint</text></g> <g><rect stroke="currentColor" fill="none" x="241.68" y="68" width="44.51999999999998" height="24" rx="5"></rect><text fill="currentColor" x="241.68" y="104" text-anchor="start">removed</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="113" x2="330" y2="113"></line><text fill="currentColor" x="0" y="127">candidate <tspan fill="currentColor" dx="8">new generation</tspan></text> <g><rect stroke="currentColor" fill="none" x="19.08" y="137" width="44.52" height="24" rx="5"></rect><text fill="currentColor" x="19.08" y="173" text-anchor="start">upload</text></g> <g><rect stroke="currentColor" fill="none" x="63.6" y="137" width="38.160000000000004" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="101.76" y="137" width="57.239999999999995" height="24" rx="5"></rect><text fill="currentColor" x="130.38" y="153" text-anchor="middle">health</text></g> <g><rect stroke="currentColor" fill="none" x="159" y="137" width="159" height="24" rx="5"></rect><text fill="currentColor" x="238.5" y="153" text-anchor="middle">promoted · serving</text></g></g> <text fill="currentColor" x="159" y="18" text-anchor="middle">promote</text> <text fill="currentColor" x="241.68" y="32" text-anchor="middle">stable endpoint verified</text></svg>

The previous deployment keeps serving until the candidate is healthy, promoted, and verified on the stable endpoint. Only then is the old one touched.

## Health checks and promotion

`healthCheck.path` gates promotion. If the candidate never becomes healthy, alchemy attempts to destroy it without changing the stable deployment. When a previous deployment exists, it keeps serving:

```typescript
const app = yield* Prisma.Compute("api", {
  project,
  path: "./app",
  port: 3000,
  healthCheck: { path: "/api/health" },
});
```

When promotion succeeds but the stable endpoint fails verification, alchemy attempts to restore the previous deployment. If recovery converges, it cleans up the failed candidate. If recovery cannot converge—or no safe rollback target exists—alchemy fails the deploy, preserves the generations needed for recovery, and retries recovery before making further cloud changes. Terminally failed deployments are never restarted or selected as rollback targets.

## Old-generation cleanup

By default the previous deployment is kept (stopped) after a successful promotion. `destroyOldDeployment: true` deletes it once the replacement is fully promoted and healthy:

```typescript
const app = yield* Prisma.Compute("api", {
  project,
  path: "./app",
  destroyOldDeployment: true,
});
```

If cleanup fails, alchemy records the pending action in state and retries it before creating another generation.

## Env-only redeploys

`skipCodeUpload: true` creates the next deployment from the currently promoted artifact. It requires an existing promoted deployment and is useful when only environment variables changed:

```typescript
const app = yield* Prisma.Compute("api", {
  project,
  skipCodeUpload: true,
  env: { FEATURE_FLAG: "on" },
});
```

## Redeploying for changed inputs

The low-level `Prisma.Deployment` resource owns no environment variables, so it cannot see an env-value change on its own. Prisma also snapshots environment variables into a deployment when the deployment is created, so changing only a value updates the platform’s variable record but never reaches the running app.

Declare the values the deployment must be recreated for with `triggers`. When the resolved value changes, the next deploy replaces the deployment (create-before-delete) even though the artifact is unchanged:

```typescript
const deployment = yield* Prisma.Deployment("web", {
  app,
  artifactPath: "./dist/app.tar.gz",
  triggers: { DATABASE_URL: Redacted.make(databaseUrl) },
  start: true,
  promote: true,
});
```

The values are folded into the deployment fingerprint as a salted hash and persisted `Redacted`; the plaintext never lands in state. `Prisma.Compute` needs no equivalent — it owns `env` and already folds those values into its own fingerprint.

## No-op convergence

Deploys are hash-driven: when the built artifact and configuration are unchanged, reconcile reuses the same healthy deployment instead of creating a new generation — repeat deploys converge to exactly one active deployment.

## Where next

- [Apps](apps.md) — builds, Effect-native services, and local dev.

Reference:

- [Compute API reference](https://alchemy.run/providers/prisma/reference/compute#compute)
- [Deployment API reference](https://alchemy.run/providers/prisma/reference/compute#deployment)
