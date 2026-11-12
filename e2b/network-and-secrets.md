---
url: https://alchemy.run/e2b/network-and-secrets
title: "Network & secrets"
description: "Limit where a sandbox can connect, and add credentials to its requests without the sandbox seeing them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A sandbox’s traffic leaves through E2B’s host, outside the microVM. That’s where E2B applies the sandbox’s **network rules**: which hosts it may call, and what to add to requests on their way out. Code in the sandbox can’t change either.

## Rules apply outside the microVM

So a **secret** never has to enter the sandbox. The sandbox sends a placeholder, and the host swaps in the real value, only on requests to the hosts you name.

<svg viewBox="0 0 479.4 406" style="max-width: 551px" role="img" aria-label="Requests leave the sandbox through E2B's host. It lets through only the hosts in allowOut, and swaps the secret's real value in for the placeholder on the way to api.github.com."><defs><marker id="alc-bd-arrow-c93h4f" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="278" y="250" width="185.39999999999998" height="140" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="250" width="232.79999999999998" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="328" x2="108.69999999999999" y2="144" marker-end="url(#alc-bd-arrow-c93h4f)"></line><text fill="currentColor" x="118.69999999999999" y="242">placeholder</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="196" x2="108.69999999999999" y2="12" marker-end="url(#alc-bd-arrow-c93h4f)"></line><text fill="currentColor" x="118.69999999999999" y="110">real token</text></g> <g><path stroke="currentColor" fill="none" d="M 108.69999999999999 328 V 374 H 274"></path><path stroke="currentColor" fill="none" d="M 269 367 l 12 14 m 0 -14 l -12 14"></path><text fill="currentColor" x="193.35" y="366" text-anchor="middle">blocked</text></g> <text fill="currentColor" x="290" y="269" text-anchor="start">not in allowOut</text> <text fill="currentColor" x="236.79999999999998" y="269" text-anchor="end">sandbox <tspan fill="currentColor" dx="8">allowOut api.github.com</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="37" text-anchor="middle">api.github.com</text> <text fill="currentColor" x="108.69999999999999" y="53" text-anchor="middle">allowed</text></g> <g><rect stroke="currentColor" fill="none" x="296" y="280" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="370.7" y="301" text-anchor="middle">example.com</text> <text fill="currentColor" x="370.7" y="317" text-anchor="middle">any other host</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="148" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="169" text-anchor="middle">E2B host</text> <text fill="currentColor" x="108.69999999999999" y="185" text-anchor="middle">adds Authorization</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="280" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="301" text-anchor="middle">agent code</text> <text fill="currentColor" x="108.69999999999999" y="317" text-anchor="middle">sends a placeholder</text></g></svg> <svg viewBox="0 0 287.59999999999997 542" style="max-width: 331px" role="img" aria-label="Requests leave the sandbox through E2B's host. It lets through only the hosts in allowOut, and swaps the secret's real value in for the placeholder on the way to api.github.com."><defs><marker id="alc-bd-arrow-x1irmy" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="416" width="185.39999999999998" height="110" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="268" width="185.39999999999998" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="360" x2="108.69999999999999" y2="160" marker-end="url(#alc-bd-arrow-x1irmy)"></line><text fill="currentColor" x="118.69999999999999" y="266">placeholder</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="108.69999999999999" y1="212" x2="108.69999999999999" y2="12" marker-end="url(#alc-bd-arrow-x1irmy)"></line><text fill="currentColor" x="118.69999999999999" y="118">real token</text></g> <g><path stroke="currentColor" fill="none" d="M 183.39999999999998 336 H 217.39999999999998 V 438 H 205.39999999999998"></path><path stroke="currentColor" fill="none" d="M 194.39999999999998 431 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="225.39999999999998" y="391">blocked</text></g> <text fill="currentColor" x="28" y="435" text-anchor="start">not in allowOut</text> <text fill="currentColor" x="189.39999999999998" y="287" text-anchor="end">sandbox <tspan fill="currentColor" x="189.39999999999998" dy="14">allowOut api.github.com</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="37" text-anchor="middle">api.github.com</text> <text fill="currentColor" x="108.69999999999999" y="53" text-anchor="middle">allowed</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="460" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="481" text-anchor="middle">example.com</text> <text fill="currentColor" x="108.69999999999999" y="497" text-anchor="middle">any other host</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="185" text-anchor="middle">E2B host</text> <text fill="currentColor" x="108.69999999999999" y="201" text-anchor="middle">adds Authorization</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="149.39999999999998" height="48" rx="8"></rect><text fill="currentColor" x="108.69999999999999" y="333" text-anchor="middle">agent code</text> <text fill="currentColor" x="108.69999999999999" y="349" text-anchor="middle">sends a placeholder</text></g></svg>

Requests leave the sandbox through E2B's host. It lets through only the hosts in allowOut, and swaps the secret's real value in for the placeholder on the way to api.github.com.

## Allow only some destinations

```typescript
const sandbox = yield* sandboxes.create({
  network: { allowOut: ["github.com", "*.github.com", "registry.npmjs.org"] },
});
```

Once `allowOut` has entries, everything else is blocked. `network: { allowOut: [] }` blocks everything. `denyOut` blocks CIDR ranges.

## Change it while the sandbox runs

```typescript
yield* sandbox.updateNetwork({ allowOut: ["api.openai.com"] });
```

The update replaces the sandbox’s rules.

## Leave through your own proxy

To send a sandbox’s traffic through a network you control, for a fixed IP address or your own inspection, give it a SOCKS5 **egress proxy**. E2B’s host connects to the proxy, so the sandbox never holds its password:

```typescript
export const MySandbox = E2B.Sandbox("MySandbox", {
  template: MyTemplate,
  network: {
    egressProxy: {
      address: "proxy.example.com:1080",
      username: "sandbox",
      password: Config.Redacted("PROXY_PASSWORD"),
      tls: { enabled: true },
    },
  },
});
```

A changed proxy updates the sandbox in place.

The proxy speaks SOCKS5 over TCP, so it runs on a host that accepts TCP connections, such as a Fly Machine or a container behind a network load balancer. A Cloudflare Worker accepts only HTTP, so it can’t be one. The proxy carries the sandbox’s TLS without opening it; to add a credential to requests, use [rules](#add-it-to-requests-on-the-way-out).

## Store a secret for the team

```typescript
import * as E2B from "@alchemy.run/e2b";
import * as Config from "effect/Config";

export const GitHubToken = E2B.Secret("GitHubToken", { value: Config.Redacted("GITHUB_TOKEN") });
```

E2B never returns a secret’s value. Changing `value` stores a new version.

## Pull values from a secrets manager

`.env` is only the default. A Stack’s `secrets` option reads the same values from a secrets manager instead, like Doppler or Infisical, so production keys never sit in a file on anyone’s machine. Pick the source per stage:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Doppler from "@alchemy.run/doppler";
import * as E2B from "@alchemy.run/e2b";
import * as Alchemy from "alchemy";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import Api from "./src/Api.ts";
import { GitHubToken } from "./src/Secrets.ts";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), E2B.providers()),
    state: Cloudflare.state(),
    secrets: ({ stage }) =>
      stage === "prod"
        ? Doppler.Secrets({ project: "my-app", config: "prd" })
        : Alchemy.Secrets.DotEnv(),
  },
  Effect.gen(function* () {
    yield* GitHubToken;
    const api = yield* Api;
    return { url: api.url.as<string>() };
  }),
);
```

Your code doesn’t change: `Config.Redacted("GITHUB_TOKEN")` reads from whichever source the stage uses. See [Secret providers](../environments/secret-providers.md).

## Add it to requests on the way out

```typescript
const github = yield* GitHubToken;
const sandbox = yield* sandboxes.create({
  network: {
    allowOut: ["api.github.com"],
    rules: {
      "api.github.com": [{ transform: { headers: { Authorization: \`Bearer ${github.ref}\` } } }],
    },
  },
});
```

`github.ref` is a placeholder (`${e2b.secrets.<name>}`). E2B swaps in the value on the host, only for requests to `api.github.com`. Code in the sandbox can’t read it.

## Create a secret per user

```typescript
const secrets = yield* E2B.CreateSecret();
// in fetch:
const token = yield* secrets.put(\`github-${user}\`, Redacted.make(userToken));
```

Provide `E2B.CreateSecretHttp`. Objects created at run time are tagged with the stack, stage and binding (`alchemy::stack`, `alchemy::stage`, `alchemy::id`) and are not deleted when the stack is destroyed. List them by those tags to clean them up. `secrets.list()` returns the ones the binding stored:

```typescript
yield* Effect.forEach(yield* secrets.list(), (secret) => secrets.delete(secret.secretId));
```

## Rotate a secret while sandboxes run

```typescript
const token = yield* E2B.WriteSecret(GitHubToken);
// in a scheduled handler:
yield* token.put(Redacted.make(fresh));
```

Sandboxes use the new version on their next request.

## Close a sandbox’s ports to the public

Sandbox ports are public unless you create the sandbox with `allowPublicTraffic: false`:

```typescript
const sandbox = yield* sandboxes.create({ network: { allowPublicTraffic: false } });
const port = yield* sandbox.getTcpPort(3000);
```

Then `port.fetch` sends the sandbox’s traffic token for you.

## Give a sandbox a cloud identity, not a key

```typescript
const sandbox = yield* sandboxes.create({
  iam: { tokens: { aws: { audience: "sts.amazonaws.com", tokenType: "JWT-SVID" } } },
});
```

The sandbox gets tokens it can exchange with your cloud, so you don’t hand it long-lived keys.
