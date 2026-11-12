---
url: https://alchemy.run/daytona/security/secrets
title: "Secrets"
description: "Give a Daytona sandbox credentials it can use but not read, shared or one per user, with values from .env or a secrets manager."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Code in a sandbox, or an agent driving it, shouldn’t be able to read your tokens. A Daytona **secret** solves that: the sandbox sees a placeholder in its environment, and Daytona’s egress proxy swaps in the real value only on requests to the hosts the secret names. The value never enters the sandbox.

## The sandbox sees only a placeholder

<svg viewBox="0 0 527.4 382" style="max-width: 607px" role="img" aria-label="The sandbox holds only a placeholder. On its way out, a request to registry.npmjs.org gets the real token; a request anywhere else keeps the placeholder."><defs><marker id="alc-bd-arrow-wx1tbo" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="218.39999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="156" width="218.39999999999998" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="125.19999999999999" y1="102" x2="125.19999999999999" y2="182" marker-end="url(#alc-bd-arrow-wx1tbo)"></line><text fill="currentColor" x="135.2" y="148">request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="125.19999999999999" y1="234" x2="125.19999999999999" y2="314" marker-end="url(#alc-bd-arrow-wx1tbo)"></line><text fill="currentColor" x="135.2" y="280">real value</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="216.39999999999998" y1="210" x2="325" y2="210" marker-end="url(#alc-bd-arrow-wx1tbo)"></line><text fill="currentColor" x="272.7" y="202" text-anchor="middle">placeholder</text></g> <text fill="currentColor" x="222.39999999999998" y="43" text-anchor="end">sandbox <tspan fill="currentColor" dx="8">never sees the value</tspan></text> <text fill="currentColor" x="222.39999999999998" y="175" text-anchor="end">Daytona</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="75" text-anchor="middle">npm install</text> <text fill="currentColor" x="125.19999999999999" y="91" text-anchor="middle">$NPM_TOKEN = placeholder</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="207" text-anchor="middle">egress proxy</text> <text fill="currentColor" x="125.19999999999999" y="223" text-anchor="middle">swaps the value in</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="318" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="339" text-anchor="middle">registry.npmjs.org</text> <text fill="currentColor" x="125.19999999999999" y="355" text-anchor="middle">gets the real token</text></g> <g><rect stroke="currentColor" fill="none" x="329" y="186" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="420.2" y="207" text-anchor="middle">any other host</text> <text fill="currentColor" x="420.2" y="223" text-anchor="middle">gets the placeholder</text></g></svg> <svg viewBox="0 0 250.39999999999998 562" style="max-width: 288px" role="img" aria-label="The sandbox holds only a placeholder. On its way out, a request to registry.npmjs.org gets the real token; a request anywhere else keeps the placeholder."><defs><marker id="alc-bd-arrow-yphvh0" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="218.39999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="172" width="218.39999999999998" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="125.19999999999999" y1="102" x2="125.19999999999999" y2="198" marker-end="url(#alc-bd-arrow-yphvh0)"></line><text fill="currentColor" x="135.2" y="156">request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="125.19999999999999" y1="250" x2="125.19999999999999" y2="346" marker-end="url(#alc-bd-arrow-yphvh0)"></line><text fill="currentColor" x="135.2" y="304">real value</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="125.19999999999999" y1="250" x2="125.19999999999999" y2="494" marker-end="url(#alc-bd-arrow-yphvh0)"></line><text fill="currentColor" x="135.2" y="378">placeholder</text></g> <text fill="currentColor" x="222.39999999999998" y="43" text-anchor="end">sandbox <tspan fill="currentColor" dx="8">never sees the value</tspan></text> <text fill="currentColor" x="222.39999999999998" y="191" text-anchor="end">Daytona</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="75" text-anchor="middle">npm install</text> <text fill="currentColor" x="125.19999999999999" y="91" text-anchor="middle">$NPM_TOKEN = placeholder</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="223" text-anchor="middle">egress proxy</text> <text fill="currentColor" x="125.19999999999999" y="239" text-anchor="middle">swaps the value in</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="371" text-anchor="middle">registry.npmjs.org</text> <text fill="currentColor" x="125.19999999999999" y="387" text-anchor="middle">gets the real token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="498" width="182.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="125.19999999999999" y="519" text-anchor="middle">any other host</text> <text fill="currentColor" x="125.19999999999999" y="535" text-anchor="middle">gets the placeholder</text></g></svg>

The sandbox holds only a placeholder. On its way out, a request to registry.npmjs.org gets the real token; a request anywhere else keeps the placeholder.

## Declare a secret for some hosts

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Config from "effect/Config";

export const NpmToken = Daytona.Secret("NpmToken", {
  value: Config.Redacted("NPM_TOKEN"),
  hosts: ["registry.npmjs.org"],
});
```

Leave out `hosts` and Daytona sends the value to any host.

## Give it to a sandbox

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";
import { NpmToken } from "./Secrets.ts";

export const DevBox = Daytona.Sandbox("DevBox", {
  snapshot: Base,
  secrets: { NPM_TOKEN: NpmToken },
});
```

`$NPM_TOKEN` in the sandbox holds the placeholder, and `npm install` from a private registry works. Changing the secrets updates the sandbox in place.

## Create a secret per user

A Worker can create a secret per user and hand it to that user’s sandbox:

```typescript
const secrets = yield* Daytona.CreateSecret();
const github = yield* secrets.put({ name: \`github-${user}\`, value: token, hosts: ["api.github.com"] });
const sandbox = yield* sandboxes.create({ secrets: [{ GITHUB_TOKEN: github.name }] });
```

The Worker’s key has `manage:secrets`. Putting the same name again replaces the value. Each secret the binding creates carries `stack/stage/id` in its description, and is not deleted when the stack is destroyed: `secrets.list()` and match the description to clean them up.

## Pull values from a secrets manager

`.env` is only the default. A Stack’s `secrets` option reads the same `Config` values from a secrets manager instead, like Doppler or Infisical, so production tokens never sit in a file on anyone’s machine. Pick the source per stage:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Daytona from "@alchemy.run/daytona";
import * as Doppler from "@alchemy.run/doppler";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { DevBox } from "./src/DevBox.ts";

export default Alchemy.Stack(
  "Sandboxes",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Daytona.providers()),
    secrets: ({ stage }) =>
      stage === "prod"
        ? Doppler.Secrets({ project: "sandboxes", config: "prd" })
        : Alchemy.Secrets.DotEnv(),
  },
  Effect.gen(function* () {
    const devBox = yield* DevBox;
    return { devBox: devBox.sandboxId };
  }),
);
```

Your code doesn’t change: `Config.Redacted("NPM_TOKEN")` reads from whichever source the stage uses. See [Secret providers](../../environments/secret-providers.md) for Doppler, Infisical, ordering several sources, and writing your own.

What a sandbox can reach at all is in [Network and previews](../networking/network.md).
