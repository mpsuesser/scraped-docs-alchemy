---
url: https://alchemy.run/modal/security/secrets
title: "Secrets"
description: "Give Modal Functions, Servers and Sandboxes API keys and passwords as Modal Secrets, as environment variables or added to requests on the way out."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A Modal **Secret** is a named set of environment variables stored in an environment. Functions, Servers and Sandboxes that attach it see its variables. A Sandbox can also leave them out of its environment entirely, and have Modal add them to its outgoing requests instead.

## Store a Secret in an environment

Give a Secret its values from `Config`, so they come from your `.env` or secrets manager and stay out of logs and state:

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Config from "effect/Config";
import { MyEnv } from "./environments.ts";

export const OpenAI = Modal.Secret("OpenAI", {
  environment: MyEnv,
  env: { OPENAI_API_KEY: Config.Redacted("OPENAI_API_KEY") },
});
```

Every deploy writes the whole set. Containers started afterwards see the new values, and running ones keep theirs.

## Pull values from a secrets manager

`.env` is only the default. A Stack’s `secrets` option reads the same `Config` values from a secrets manager instead, like Doppler or Infisical, so production keys never sit in a file on anyone’s machine. Pick the source per stage:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Doppler from "@alchemy.run/doppler";
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { OpenAI } from "./src/secrets.ts";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Layer.mergeAll(Cloudflare.providers(), Modal.providers()),
    state: Cloudflare.state(),
    secrets: ({ stage }) =>
      stage === "prod"
        ? Doppler.Secrets({ project: "my-app", config: "prd" })
        : Alchemy.Secrets.DotEnv(),
  },
  Effect.gen(function* () {
    const openai = yield* OpenAI;
    return { secret: openai.name };
  }),
);
```

`Config.Redacted("OPENAI_API_KEY")` doesn’t change. See [Secret providers](../../environments/secret-providers.md).

## Attach it as environment variables

A Function or Server lists the Secrets its containers see:

```typescript
export default class Embed extends Modal.Function<Embed>()(
  "Embed",
  { main: import.meta.url, app: Inference, secrets: [OpenAI] },
  Effect.gen(function* () {
    const key = yield* Config.Redacted("OPENAI_API_KEY");
    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }),
) {}
```

A sandbox your code creates takes them by id. Read the id while the Worker is constructed, and pass it in the handler:

```typescript
// construction phase
const openaiId = yield* (yield* OpenAI).secretId;
// in a handler
const openai = { secretId: yield* openaiId };
const sandbox = yield* sandboxes.create({ secrets: [openai] });
```

## Add it to requests, never into the sandbox

Code an agent writes can print its environment. To keep a key from it, leave the Secret out of `secrets` and give the sandbox an **outbound policy**: for requests to the hosts you name, Modal replaces headers with the Secret’s value on their way out.

<svg viewBox="0 0 502 250" style="max-width: 577px" role="img" aria-label="The agent's request leaves the sandbox without a key. Modal's egress adds the Authorization header from the Secret, only for api.openai.com."><defs><marker id="alc-bd-arrow-edtw2a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="172" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="170" y1="78" x2="305" y2="78" marker-end="url(#alc-bd-arrow-edtw2a)"></line><text fill="currentColor" x="239.5" y="70" text-anchor="middle">request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="377" y1="102" x2="377" y2="182" marker-end="url(#alc-bd-arrow-edtw2a)"></line><text fill="currentColor" x="387" y="148">+ Authorization</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">session-sam <tspan fill="currentColor" dx="8">Sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="136" height="48" rx="8"></rect><text fill="currentColor" x="102" y="75" text-anchor="middle">agent code</text> <text fill="currentColor" x="102" y="91" text-anchor="middle">no key</text></g> <g><rect stroke="currentColor" fill="none" x="309" y="54" width="136" height="48" rx="8"></rect><text fill="currentColor" x="377" y="75" text-anchor="middle">Modal egress</text> <text fill="currentColor" x="377" y="91" text-anchor="middle">outbound policy</text></g> <g><rect stroke="currentColor" fill="none" x="309" y="186" width="136" height="48" rx="8"></rect><text fill="currentColor" x="377" y="215" text-anchor="middle">api.openai.com</text></g></svg> <svg viewBox="0 0 227 414" style="max-width: 261px" role="img" aria-label="The agent's request leaves the sandbox without a key. Modal's egress adds the Authorization header from the Secret, only for api.openai.com."><defs><marker id="alc-bd-arrow-pwlcsf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="172" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="102" y1="102" x2="102" y2="198" marker-end="url(#alc-bd-arrow-pwlcsf)"></line><text fill="currentColor" x="112" y="156">request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="102" y1="250" x2="102" y2="346" marker-end="url(#alc-bd-arrow-pwlcsf)"></line><text fill="currentColor" x="112" y="304">+ Authorization</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">session-sam <tspan fill="currentColor" dx="8">Sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="136" height="48" rx="8"></rect><text fill="currentColor" x="102" y="75" text-anchor="middle">agent code</text> <text fill="currentColor" x="102" y="91" text-anchor="middle">no key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="136" height="48" rx="8"></rect><text fill="currentColor" x="102" y="223" text-anchor="middle">Modal egress</text> <text fill="currentColor" x="102" y="239" text-anchor="middle">outbound policy</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="136" height="48" rx="8"></rect><text fill="currentColor" x="102" y="379" text-anchor="middle">api.openai.com</text></g></svg>

The agent's request leaves the sandbox without a key. Modal's egress adds the Authorization header from the Secret, only for api.openai.com.

```typescript
const sandbox = yield* sandboxes.create({
  outboundPolicy: {
    routes: [
      {
        https: {
          domains: ["api.openai.com"],
          headerReplacements: [{ secret: openai, headers: { Authorization: "Bearer $OPENAI_API_KEY" } }],
        },
      },
      { any: {} },
    ],
  },
});
```

`$OPENAI_API_KEY` names a variable in the Secret. The last route, `{ any: {} }`, lets every other request through unchanged; leave it out to allow only the routes you list.

## Create a Secret per user

A user’s own token doesn’t belong in your Stack. Bind `CreateSecret` to an environment and put one by name while the Worker runs:

```typescript
const secrets = yield* Modal.CreateSecret(MyEnv);
const github = yield* secrets.put(\`github-${user}\`, { GITHUB_TOKEN: token });
const sandbox = yield* sandboxes.create({ secrets: [github] });
```

Provide `Modal.CreateSecretHttp`. The Worker’s service user gets `contributor` in the environment. Secrets created this way carry the stack, stage and binding in their name, and stay when the Stack is destroyed.

## Change a value while it runs

`WriteSecret` replaces the values of one Secret the Stack declares, like a token you rotate on a schedule. Modal never returns a Secret’s values, so there is no read binding:

```typescript
const openai = yield* Modal.WriteSecret(OpenAI);
yield* openai.put({ OPENAI_API_KEY: next });
```

## Give a sandbox a cloud identity, not a key

For a cloud that trusts an OIDC issuer, like AWS or Google Cloud, a sandbox needs no stored key at all. Create it with `includeOidcIdentityToken`, and Modal sets `MODAL_IDENTITY_TOKEN`: a token Modal signs as `https://oidc.modal.com`, whose subject names the workspace, environment and app.

```typescript
const sandbox = yield* sandboxes.create({
  image: "amazon/aws-cli",
  includeOidcIdentityToken: true,
});
const role = yield* sandbox.exec(
  \`aws sts assume-role-with-web-identity --role-arn ${roleArn} --role-session-name sandbox --web-identity-token "$MODAL_IDENTITY_TOKEN"\`,
);
```

The role trusts Modal’s issuer and matches the token’s subject, so only your workspace’s sandboxes can assume it, and the credentials it returns expire on their own. `includeX509Svid` serves a certificate that names the sandbox instead, for AWS IAM Roles Anywhere.
