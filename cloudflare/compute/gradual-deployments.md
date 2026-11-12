---
url: https://alchemy.run/cloudflare/compute/gradual-deployments
title: "Gradual deployments"
description: "Roll out Worker deploys incrementally — upload preview versions, canary a percentage of live traffic, ramp to 100%, pin users to a version during the rollout, and verify which version served each request."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Every deploy of a Worker uploads an immutable [version](https://developers.cloudflare.com/workers/configuration/versions-and-deployments/), a snapshot of code, bindings, and compatibility settings. A *deployment* routes traffic across up to two versions by percentage. By default alchemy deploys each new version at 100%. The `version` prop unlocks the other shapes: gradual rollouts, canaries, and traffic-free version uploads. For branch and pull-request testing, use [Worker Previews](previews.md) (`preview.of`) instead of a zero-traffic version.

## Roll out a deploy gradually

Set `version.traffic` below 100 and the deploy splits traffic instead of cutting over:

```typescript
yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  version: {
    // 10% to the new version, 90% stays on the live one
    traffic: 10,
    // optional annotations, shown in the dashboard & \`wrangler versions list\`
    message: "new search index",
    tag: process.env.GITHUB_SHA,
  },
});
```

Each request is routed independently by percentage. Raise the number and re-deploy to ramp; remove the prop (or set `100`) to promote.

<svg viewBox="0 0 698.4 124" style="max-width: 803px" role="img" aria-label="Each deploy moves the split. Until promotion, both versions serve live traffic, chosen per request."><line stroke-opacity="0.2" stroke="currentColor" x1="294.4" y1="24" x2="294.4" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="462.4" y1="24" x2="462.4" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="574.4" y1="24" x2="574.4" y2="118"></line><g><text fill="currentColor" x="4" y="50">version 12</text> <text fill="currentColor" x="4" y="64">live</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="40" width="167.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="210.39999999999998" y="56" text-anchor="middle">100%</text></g> <g><rect stroke="currentColor" fill="none" x="294.4" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="378.4" y="56" text-anchor="middle">90%</text></g> <g><rect stroke="currentColor" fill="none" x="462.4" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="518.4" y="56" text-anchor="middle">50%</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="698.4" y2="74"></line><text fill="currentColor" x="4" y="94">version 13</text> <text fill="currentColor" x="4" y="108">new search index</text> <g><rect stroke="currentColor" fill="none" x="294.4" y="84" width="168" height="24" rx="5"></rect><text fill="currentColor" x="378.4" y="100" text-anchor="middle">10%</text></g> <g><rect stroke="currentColor" fill="none" x="462.4" y="84" width="112" height="24" rx="5"></rect><text fill="currentColor" x="518.4" y="100" text-anchor="middle">50%</text></g> <g><rect stroke="currentColor" fill="none" x="574.4" y="84" width="112" height="24" rx="5"></rect><text fill="currentColor" x="630.4" y="100" text-anchor="middle">100%</text></g></g> <text fill="currentColor" x="294.4" y="18" text-anchor="middle">traffic: 10</text> <text fill="currentColor" x="462.4" y="18" text-anchor="middle">traffic: 50</text> <text fill="currentColor" x="574.4" y="18" text-anchor="middle">promote</text></svg> <svg viewBox="0 0 330 158" style="max-width: 379px" role="img" aria-label="Each deploy moves the split. Until promotion, both versions serve live traffic, chosen per request."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="38" x2="95.39999999999999" y2="152"></line><line stroke-opacity="0.2" stroke="currentColor" x1="190.79999999999998" y1="38" x2="190.79999999999998" y2="152"></line><line stroke-opacity="0.2" stroke="currentColor" x1="254.4" y1="38" x2="254.4" y2="152"></line><g><text fill="currentColor" x="0" y="58">version 12 <tspan fill="currentColor" dx="8">live</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="68" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="84" text-anchor="middle">100%</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="68" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="84" text-anchor="middle">90%</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="68" width="63.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="222.6" y="84" text-anchor="middle">50%</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="98" x2="330" y2="98"></line><text fill="currentColor" x="0" y="112">version 13 <tspan fill="currentColor" dx="8">new search index</tspan></text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="122" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="138" text-anchor="middle">10%</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="122" width="63.60000000000002" height="24" rx="5"></rect><text fill="currentColor" x="222.6" y="138" text-anchor="middle">50%</text></g> <g><rect stroke="currentColor" fill="none" x="254.4" y="122" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="286.2" y="138" text-anchor="middle">100%</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">traffic: 10</text> <text fill="currentColor" x="190.79999999999998" y="18" text-anchor="middle">traffic: 50</text> <text fill="currentColor" x="254.4" y="32" text-anchor="middle">promote</text></svg>

Each deploy moves the split. Until promotion, both versions serve live traffic, chosen per request.

The first deploy of a script always goes to 100%, since there is nothing to split with. `traffic: 0` uploads the version without deploying it (the equivalent of `wrangler versions upload`).

## Percentages replace, they never add

A deployment is a complete routing table, and each versioned deploy splits its new version against the *stable* side of the current one, the majority holder:

```typescript
version: { traffic: 10 }  // deploys: v2 10%, v1 90%
version: { traffic: 20 }  // deploys: v3 20%, v1 80% — v2 drops out
```

The dropped version keeps existing as an upload, but it gets no traffic and a version override can no longer reach it. With unchanged code, v3 is identical to v2 and this *is* the ramp from 10 to 20; with changed code, you have swapped canaries. Either way there is one stable version and at most one canary per script, until you promote.

## Ramp from CI

Read the percentage from the environment so a ramp step is a workflow re-run, not a code change:

```typescript
yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  version: {
    traffic: process.env.API_TRAFFIC ? Number(process.env.API_TRAFFIC) : 100,
  },
});
```

```yaml
on:
  workflow_dispatch:
    inputs:
      traffic:
        description: "Traffic % for the new version"
        default: "100"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: bun alchemy deploy
        env:
          API_TRAFFIC: ${{ inputs.traffic }}
```

Deploy at `10`, watch your metrics, re-run at `50`, promote at the default `100`. Run every step from the same ref. A re-run from a moving branch ships whatever landed since, which is a new rollout, not a ramp.

## Roll back

Rolling back is another deploy: re-deploy the previous code and it goes out as a new version at 100%. Versions are immutable snapshots, so the previous behavior comes back exactly, bindings and compatibility settings included.

## Canary another stage’s Worker

Set `version.parent` to upload this Worker’s code as a version of *another* Worker’s script instead of creating a script of its own:

```typescript
const parent = yield* Cloudflare.Worker.ref("Api", { stage: "prod" });

yield* Cloudflare.Worker("ApiCanary", {
  main: "./src/api.ts",
  version: { parent, traffic: 10 },
});
```

Destroying the canary restores 100% of traffic to the live version.

## Upload a version without routing traffic

With `traffic: 0` the version is uploaded and reachable at its Version URL, but it is not in the live deployment. Use this to inspect a specific upload before a gradual rollout. For branch and pull-request testing — isolated Durable Objects, Preview-specific bindings, custom domain Preview URLs — use [Worker Previews](previews.md) (`preview.of`) instead.

```typescript
yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  version: { traffic: 0, tag: process.env.GITHUB_SHA },
});
```

`version.parent` with no `traffic` still uploads a Version URL of another stage’s Worker (the previous PR-preview path). Prefer `preview.of` for that job going forward.

### Version, Preview, or stage?

A Preview is a named copy of a Worker with isolated Durable Object state. A version rides the parent’s script and version history. A stage is a fully independent stack. Use a stage to preview infrastructure changes; use a Preview to preview code; use a version to canary or inspect an upload. See [Worker Previews](previews.md).

## Smoke test before shifting traffic

The `Cloudflare-Workers-Version-Overrides` request header pins a request to a specific version in the current deployment:

```typescript
const api = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  // 1% — the smallest deployment an override can reach. traffic: 0
  // stays out of the deployment entirely (preview URL only).
  version: { traffic: 1 },
});

return { url: api.url.as<string>(), versionId: api.versionId };
```

```sh
curl -s https://api.example.com/health \
  -H 'Cloudflare-Workers-Version-Overrides: my-api="<versionId>"'
```

The dictionary is keyed by the *physical script name*; an unknown version or malformed header falls back to the percentages. Overrides also apply to `fetch()` -based service binding calls. Forward the incoming request to keep cooperating Workers on matching versions. If the smoke test fails, re-deploy the last good ref at the default 100 ([Roll back](#roll-back)).

## Keep users on one version

Percentages route each request independently, so one user can bounce between versions mid-rollout. The `Cloudflare-Workers-Version-Key` header pins them: Cloudflare hashes the key and assigns a version deterministically from the percentages. Declare the key source with `version.affinity`:

```typescript
yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  domain: "api.example.com",
  version: {
    traffic: 10,
    // sticky by session cookie, falling back to sticky IP
    affinity: { cookie: "session_id", ip: true },
  },
});
```

Alchemy provisions a transform rule on each zone the Worker serves on (custom domains and routes), scoped to the Worker’s hostnames, that fills the header from the cookie, and for requests that don’t carry the cookie yet, from the client IP. The rule updates in place as the config changes and is removed when `affinity` is removed or the Worker is destroyed. On a [canary of another stage’s Worker](#canary-another-stages-worker) (`version.parent`), the rule lands on the parent’s zones, pinning users across the canary split.

Besides `cookie`, the key can come from a request `header` (a tenant or user id your edge already stamps), the client IP alone (`{ ip: true }`), or, for anything else like a JWT claim via API Shield, a raw [Rules-language](https://developers.cloudflare.com/ruleset-engine/rules-language/) expression:

```typescript
version: {
  traffic: 10,
  affinity: { key: \`http.request.jwt.claims["<token-config-id>"][0]\` },
}
```

Transform rules only see zone traffic; on the bare `workers.dev` URL, clients send the header themselves:

```sh
curl -s https://api.example.com -H 'Cloudflare-Workers-Version-Key: user-123'
```

Pinned users only move forward as percentages rise, never back.

For full control over the rule, such as custom match conditions, keys composed from several fields, or rules spanning Workers, drop down to a manual `Ruleset` in the same phase, which is exactly what the sugar manages for you:

```typescript
const zone = yield* Cloudflare.Zone.Zone("Zone", { name: "example.com" });

yield* Cloudflare.Ruleset.Ruleset("VersionAffinity", {
  zone,
  phase: "http_request_late_transform",
  rules: [
    {
      description: "Pin users to a Worker version by session cookie",
      expression: \`len(http.request.cookies["session_id"]) > 0\`,
      action: "rewrite",
      actionParameters: {
        headers: {
          "Cloudflare-Workers-Version-Key": {
            operation: "set",
            expression: \`http.request.cookies["session_id"][0]\`,
          },
        },
      },
    },
  ],
});
```

## Verify which version served a request

The `version_metadata` binding exposes the running version’s `id`, `tag`, and `timestamp`, tying every response, log, and error report to a deploy:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const versionMetadata = yield* Cloudflare.Workers.VersionMetadata();

    return {
      fetch: Effect.gen(function* () {
        // \`tag\` is whatever version.tag was set to — e.g. the git SHA
        const { id, tag } = yield* versionMetadata;
        return yield* HttpServerResponse.json({ ok: true, version: id, tag });
      }),
    };
  }).pipe(Effect.provide(Cloudflare.Workers.VersionMetadataBinding)),
);
```

See [Version metadata](workers.md#versions) for the async-Worker variant.

## What a version carries

A version snapshots code, bindings, compatibility settings, and cache configuration. Script-level settings (routes, custom domains, crons, tags, observability, logpush, placement, limits, and the workers.dev subdomain) apply immediately to *all* versions: they are rejected on a version worker, and during a gradual rollout they keep their live values until the next full deploy.

Anything the Worker *calls*, Durable Objects, Workflows, and service bindings, sits outside the version too, so both versions talk to it during the rollout. Ship those changes expand-and-contract:

1. Deploy the new DO handler / Workflow / RPC method at 100%, alongside the old one.
2. Ramp the caller gradually; the old version keeps calling the old path.
3. After promotion, remove the old path.

The same goes for data: both versions share the same KV, D1, and DO storage, so schema changes must stay compatible until the rollout completes.

Platform limits to plan around:

- A deployment splits between at most two versions, and only the last 100 uploaded versions are deployable.
- Each Durable Object is assigned one version by the percentages, so a single object never splits across versions. Durable Object class changes (new, renamed, deleted or moved classes) cannot ride a rollout (deploy them at 100% first), and a version worker cannot host DO or Workflow classes. Host them on the parent and reference them [cross-script](cross-worker-durable-object.md).
- Workers that implement Durable Objects do not get preview URLs.
- [Workers Cache](cache.md) is per-version by default, so every version starts cold; `crossVersionCache: true` shares it across versions.

## Containers roll out by instance, not by request

[Containers](containers.md) shift gradually by replacing running *instances* with the new image in steps. There is no request-level split between two images:

```typescript
export class Api extends Cloudflare.Container<Api>()("Api", {
  main: import.meta.url,
  instances: 4,
  maxInstances: 4,
  // replace 25% of instances per step, advancing as they come up
  // healthy; omit \`rollout\` to replace all instances at once
  rollout: { strategy: "rolling", stepPercentage: 25 },
}) {}
```

Alchemy creates the rollout automatically whenever the image or configuration changes. The fronting Worker cuts over immediately while instances roll, so keep the Worker-to-container protocol compatible across both images until the rollout completes. See [Rollouts](containers.md#rollouts) in the Containers guide.

## Durable Objects roll out by object

Each Durable Object runs one version. Raising the traffic moves more objects onto the new version, and each one waits to go idle before it switches. See [Gradual deployments assign objects to versions](code-updates.md#gradual-deployments-assign-objects-to-versions).

## Where next

- [Workers](workers.md) — the full Worker reference, including the `version` prop and URLs.
- [Custom domains](../networking/custom-domains.md) — serve the rollout from your own hostnames.
- [Workers Cache](cache.md) — edge caching and the cross-version cache.
- [Cloudflare’s gradual deployments docs](https://developers.cloudflare.com/workers/configuration/versions-and-deployments/gradual-deployments/) — the underlying platform feature.
