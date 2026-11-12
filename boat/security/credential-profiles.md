---
url: https://alchemy.run/boat/security/credential-profiles
title: "Credential profiles"
description: "Boat credential profiles — the AI keys and API secrets a sandbox gets, and the proxy that keeps the real ones out of it."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A **credential profile** is a named set of AI credentials that Boat stores encrypted: API keys for Anthropic, OpenAI and other providers, secrets for any other HTTPS API, and the Claude or ChatGPT sign-ins you make on the dashboard. A sandbox gets the profile its [environment](environments.md) names. Boat never returns a secret once it’s stored.

## Declare a profile

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Config from "effect/Config";

export const MyProfile = Boat.CredentialProfile("MyProfile", {
  anthropic: Config.Redacted("ANTHROPIC_API_KEY"),
  openai: Config.Redacted("OPENAI_API_KEY"),
});
```

Each provider has its own prop: `anthropic`, `openai`, `openrouter`, `llmgateway`, `deepseek`, `moonshot`, `mistral`, `xai` and `opencode`.

The props are the whole list. A deploy sets a key whose value changed, and removes an API key the props no longer name. Sign-ins and Bedrock are connected on the dashboard’s **Agents & Proxying** page, and a deploy leaves them alone.

## Give sandboxes the profile

Name the profile on an environment, and start sandboxes on it:

```typescript
import * as Boat from "@alchemy.run/boat";
import { MyProfile } from "./MyProfile.ts";

export const MyEnv = Boat.Environment("MyEnv", {
  credentialProfile: MyProfile,
});

export const Box = Boat.Sandbox("Box", { environment: MyEnv });
```

`Box` starts with `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` set. An environment that names no profile passes no AI credentials.

Boat refuses to delete a profile an environment still names, so a destroy deletes the environment first.

## Add a secret for any other API

`apis` holds a secret for any HTTPS API, with the base URL it’s sent to. The key of each entry is the variable the sandbox gets:

```typescript
import * as Boat from "@alchemy.run/boat";
import * as Config from "effect/Config";

export const MyProfile = Boat.CredentialProfile("MyProfile", {
  anthropic: Config.Redacted("ANTHROPIC_API_KEY"),
  apis: {
    STRIPE_KEY: {
      key: Config.Redacted("STRIPE_KEY"),
      baseUrl: "https://api.stripe.com/v1",
    },
  },
});
```

A sandbox gets `STRIPE_KEY` and `STRIPE_KEY_BASE_URL`. Code in the sandbox sends its requests to `$STRIPE_KEY_BASE_URL`, never to the API’s own address:

```sh
curl "$STRIPE_KEY_BASE_URL/charges" \
  -H "Authorization: Bearer $STRIPE_KEY"
```

The name is `A-Z`, `0-9` and `_`, up to 64 characters. Provider names like `ANTHROPIC_*` and `AWS_*`, and names ending in `_BASE_URL`, are reserved. The base URL is HTTPS on the default port, with a public DNS name.

## Keep real keys out of the sandbox

A sandbox normally holds the real keys, and anyone with a shell in it can copy them. With Boat’s **AI credentials proxy** on, the sandbox gets a placeholder for each key and the proxy’s URL for each base URL. Boat swaps the real key in as the request passes through.

<svg viewBox="0 0 683.8 268" style="max-width: 786px" role="img" aria-label="With the proxy on, the sandbox holds a placeholder and the proxy's URL. The real key stays in the profile, and Boat adds it to each request on its way to the provider."><defs><marker id="alc-bd-arrow-cfmurd" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="156" width="172.2" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="264.79999999999995" y="24" width="172.2" height="228" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="170.2" y1="210" x2="278.79999999999995" y2="210" marker-end="url(#alc-bd-arrow-cfmurd)"></line><text fill="currentColor" x="226.49999999999997" y="202" text-anchor="middle">placeholder</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="350.9" y1="102" x2="350.9" y2="182" marker-end="url(#alc-bd-arrow-cfmurd)"></line><text fill="currentColor" x="360.9" y="148">key</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418.99999999999994" y1="210" x2="527.5999999999999" y2="210" marker-end="url(#alc-bd-arrow-cfmurd)"></line><text fill="currentColor" x="475.29999999999995" y="202" text-anchor="middle">real key</text></g> <text fill="currentColor" x="28" y="175" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">no real keys</tspan></text> <text fill="currentColor" x="276.79999999999995" y="43" text-anchor="start">Boat</text> <g><rect stroke="currentColor" fill="none" x="282.79999999999995" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="350.9" y="75" text-anchor="middle">MyProfile</text> <text fill="currentColor" x="350.9" y="91" text-anchor="middle">has the real key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="207" text-anchor="middle">claude</text> <text fill="currentColor" x="102.1" y="223" text-anchor="middle">has a placeholder</text></g> <g><rect stroke="currentColor" fill="none" x="282.79999999999995" y="186" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="350.9" y="207" text-anchor="middle">AI proxy</text> <text fill="currentColor" x="350.9" y="223" text-anchor="middle">swaps the key in</text></g> <g><rect stroke="currentColor" fill="none" x="531.5999999999999" y="186" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="599.6999999999999" y="207" text-anchor="middle">Anthropic</text> <text fill="currentColor" x="599.6999999999999" y="223" text-anchor="middle">gets the real key</text></g></svg> <svg viewBox="0 0 400.2 414" style="max-width: 460px" role="img" aria-label="With the proxy on, the sandbox holds a placeholder and the proxy's URL. The real key stays in the profile, and Boat adds it to each request on its way to the provider."><defs><marker id="alc-bd-arrow-z49wm3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="212" y="24" width="172.2" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="172" width="368.2" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="298.1" y1="102" x2="298.1" y2="198" marker-end="url(#alc-bd-arrow-z49wm3)"></line><text fill="currentColor" x="308.1" y="156">placeholder</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="170.2" y1="226" x2="226" y2="226" marker-end="url(#alc-bd-arrow-z49wm3)"></line><text fill="currentColor" x="200.1" y="218" text-anchor="middle">key</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="298.1" y1="250" x2="298.1" y2="346" marker-end="url(#alc-bd-arrow-z49wm3)"></line><text fill="currentColor" x="308.1" y="304">real key</text></g> <text fill="currentColor" x="224" y="43" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">no real keys</tspan></text> <text fill="currentColor" x="28" y="191" text-anchor="start">Boat</text> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="223" text-anchor="middle">MyProfile</text> <text fill="currentColor" x="102.1" y="239" text-anchor="middle">has the real key</text></g> <g><rect stroke="currentColor" fill="none" x="230" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="298.1" y="75" text-anchor="middle">claude</text> <text fill="currentColor" x="298.1" y="91" text-anchor="middle">has a placeholder</text></g> <g><rect stroke="currentColor" fill="none" x="230" y="202" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="298.1" y="223" text-anchor="middle">AI proxy</text> <text fill="currentColor" x="298.1" y="239" text-anchor="middle">swaps the key in</text></g> <g><rect stroke="currentColor" fill="none" x="230" y="350" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="298.1" y="371" text-anchor="middle">Anthropic</text> <text fill="currentColor" x="298.1" y="387" text-anchor="middle">gets the real key</text></g></svg>

With the proxy on, the sandbox holds a placeholder and the proxy's URL. The real key stays in the profile, and Boat adds it to each request on its way to the provider.

A placeholder is worth nothing outside Boat, and a copied one stops working when the sandbox stops: every start gets new ones. The proxy sends each provider key only to its own provider, and only in the `Authorization` or `x-api-key` header.

The proxy is one switch for a whole account. `Boat.AiProxy` turns it on:

```typescript
import * as Boat from "@alchemy.run/boat";

export const Proxy = Boat.AiProxy("Proxy");
```

Boat bills the proxy: $1 for each day it’s on, and $1 per 1,000 proxied requests. With it on, every sandbox on one of the account’s profiles gets placeholders from its next start. Destroying the Stack puts the switch back the way it was.

Bedrock credentials sign each request, so there’s nothing to swap. Boat keeps them out of sandboxes while the proxy is on.

## Give users’ sandboxes AI without the keys

A `safeForThirdParties` environment passes nothing of your account, AI credentials included. `proxyAiCredentials` passes them anyway, as placeholders:

```typescript
import * as Boat from "@alchemy.run/boat";
import { MyProfile } from "./MyProfile.ts";
import { Proxy } from "./Proxy.ts";

export const Tenants = Boat.Environment("Tenants", {
  safeForThirdParties: true,
  credentialProfile: MyProfile,
  proxyAiCredentials: Proxy,
});
```

Someone with a shell in a `Tenants` sandbox can run an agent, and can’t take your key. Passing the proxy itself means the switch is turned on before the environment asks for it: Boat rejects `proxyAiCredentials` while the proxy is off.

## Use the profile you set up on the dashboard

Every account has a profile named `default`, which holds the sign-ins and keys you connect on the dashboard. `lookup` finds it by name, and the Stack never changes or deletes it:

```typescript
import * as Boat from "@alchemy.run/boat";

export const MyEnv = Boat.Environment("MyEnv", {
  credentialProfile: Boat.CredentialProfile.lookup({ name: "default" }),
});
```

This is how a sandbox gets your Claude or ChatGPT subscription, which only a browser sign-in can connect.

A profile is yours, or an organization’s. When the Stack [works in an organization](../account-and-keys.md#work-in-an-organization), the profiles it declares belong to the organization: every member’s sandboxes can use them, and only owners change them. Name one on an environment of the same organization.

The proxy follows the same rule. An organization has its own switch, which only an owner changes, and `Boat.AiProxy` in a Stack that works in the organization is that switch.

## Pull keys from a secrets manager

`Config.Redacted("ANTHROPIC_API_KEY")` reads `.env` by default. A Stack’s `secrets` option reads the same name from a secrets manager like Doppler or Infisical instead, so a production key never sits in a file on anyone’s machine. See [Secret providers](../../environments/secret-providers.md).
