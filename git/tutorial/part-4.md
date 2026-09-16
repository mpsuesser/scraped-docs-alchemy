---
url: https://alchemy.run/git/tutorial/part-4
title: "Part 4: Give users their own credentials"
description: "Replace the shared credential with Better Auth accounts and individual Git API keys."
access_date: 2026-09-16T06:33:56.799Z
current_date: 2026-09-16T06:33:56.799Z
---

One shared credential cannot distinguish one caller from another. Replace it with Better Auth accounts and API keys, then use the authenticated user’s ID when checking access to a named repository.

Keep the host and storage from [Part 3](part-3.md). Existing repositories remain in place. This part creates new repositories under user IDs; it does not transfer the existing `acme/web` repository to a user.

## Install Better Auth

```sh
bun add @alchemy.run/better-auth better-auth @better-auth/api-key
```

Better Auth will manage accounts and credentials. Git will continue to store repositories.

## Store accounts in D1

Create `src/auth.ts`:

```typescript
import { BetterAuth } from "@alchemy.run/better-auth";
import * as Cloudflare from "alchemy/Cloudflare";

export const AuthDb = Cloudflare.D1.Database("AuthDb");

export const Auth = BetterAuth({
  basePath: "/api/auth",
  emailAndPassword: { enabled: true },
});
```

This enables email/password accounts under `/api/auth`. The host will supply D1 as the database implementation later in this part.

## Enable individual API keys

```typescript
import { BetterAuth } from "@alchemy.run/better-auth";
import { apiKey } from "@better-auth/api-key";
```

Register the plugin:

```typescript
emailAndPassword: { enabled: true },
  plugins: [apiKey()],
});
```

A signed-in user can now mint an API key for a Git client. That key replaces the shared credential in the HTTP Basic password field.

## Set a request allowance for Git

```typescript
plugins: [apiKey()],
plugins: [apiKey({
  rateLimit: { timeWindow: 60_000, maxRequests: 1_000 },
})],
```

A push or clone makes multiple authenticated HTTP requests. Set an explicit allowance of 1,000 requests per minute for the keys created in this tutorial; the plugin’s default of ten requests per day is too small for these steps.

## Represent the caller

Create `src/session.ts`:

```typescript
import * as Context from "effect/Context";

export class Session extends Context.Service<
  Session,
  { readonly user: { readonly id: string } | null }
>()("app/Session") {}
```

This service carries the caller through one request. A user has an ID; `null` represents an anonymous caller reading a public repository.

Create `src/credentials.ts`:

```typescript
import * as Effect from "effect/Effect";
import { Auth } from "./auth.ts";

export const ResolveUser = Effect.gen(function* () {
  const auth = yield* Auth;
  return Effect.gen(function* () {
    const session = yield* auth.getSession().pipe(
      Effect.catchTag("BetterAuthApiError", () => Effect.succeed(null)),
    );
    return session ? { id: session.user.id.toLowerCase() } : null;
  });
});
```

As with `PublicRead`, the outer effect acquires a dependency and returns a check that runs per request. Better Auth reads the request’s session cookie. Lowercase the ID because Git repository owners are normalized to lowercase.

## Accept an API key from Git

Add the HTTP Basic decoder imports:

```typescript
import * as Effect from "effect/Effect";
import * as Redacted from "effect/Redacted";
import * as HttpApiBuilder from "effect/unstable/httpapi/HttpApiBuilder";
import * as HttpApiSecurity from "effect/unstable/httpapi/HttpApiSecurity";
```

Check for an API key before looking for a session cookie:

```typescript
const auth = yield* Auth;
return Effect.gen(function* () {
  const { password } = yield* HttpApiBuilder.securityDecode(HttpApiSecurity.basic);
  const key = Redacted.value(password);
  if (key !== "") {
    const verified = yield* auth.api.verifyApiKey({ body: { key } }).pipe(
      Effect.catchTag("BetterAuthApiError", () =>
        Effect.succeed({ valid: false as const, key: null }),
      ),
    );
    return verified.valid && verified.key
      ? { id: verified.key.referenceId.toLowerCase() }
      : null;
  }
  const session = yield* auth.getSession().pipe(
```

The key identifies its owner. An invalid key produces an anonymous caller, so it cannot grant access to a private repository or a write operation.

## Authorize the identified user

Replace `src/middleware.ts` with:

```typescript
import { RuntimeContext } from "alchemy";
import * as Effect from "effect/Effect";
import * as HttpRouter from "effect/unstable/http/HttpRouter";
import * as HttpServerResponse from "effect/unstable/http/HttpServerResponse";
import { ResolveUser } from "./credentials.ts";
import { PublicRead } from "./public-read.ts";
import { Session } from "./session.ts";

export const Authentication = HttpRouter.middleware<{ provides: Session }>()(
  Effect.gen(function* () {
    const resolveUser = yield* ResolveUser;
    const publicRead = yield* PublicRead;
    return (httpEffect) =>
      Effect.gen(function* () {
        const user = yield* resolveUser;
        const { owner } = yield* HttpRouter.params;
        const own = owner === undefined || owner.toLowerCase() === user?.id;
        if ((user !== null && own) || (yield* publicRead)) {
          return yield* Effect.provideService(httpEffect, Session, { user });
        }
        return HttpServerResponse.jsonUnsafe(
          { _tag: "Unauthorized" },
          { status: 401, headers: { "www-authenticate": 'Basic realm="git"' } },
        );
      }).pipe(Effect.provide(RuntimeContext.phantom));
  }),
);
```

For routes naming an owner, the user’s ID must match that owner unless the request is a public read. The middleware also provides `Session` for application handlers to use in Part 5. Routes without an owner parameter require a signed-in user; they do not acquire automatic tenant filtering. This tutorial creates repositories under the caller’s ID explicitly.

## Serve Better Auth’s routes

Add the auth imports to `src/host.ts`:

```typescript
import * as HttpRouter from "effect/unstable/http/HttpRouter";
import { HttpServerRequest } from "effect/unstable/http/HttpServerRequest";
import { CloudflareD1 } from "@alchemy.run/better-auth/CloudflareD1";
import { Auth, AuthDb } from "./auth.ts";
```

Initialize Better Auth beside the Git router:

```typescript
Effect.gen(function* () {
  const auth = yield* Auth;
  const fetch = yield* HttpRouter.toHttpEffect(
```

Send `/api/auth` requests to Better Auth. They must be reachable before a user has signed in:

```typescript
return { fetch };
return {
  fetch: Effect.gen(function* () {
    const request = yield* HttpServerRequest;
    const path = request.url.split("?")[0];
    if (path === "/api/auth" || path?.startsWith("/api/auth/")) {
      return yield* auth.fetch;
    }
    return yield* fetch;
  }),
};
```

Supply the D1 database on the Worker’s initialization effect:

```typescript
}),
  }).pipe(Effect.provide(CloudflareD1(AuthDb))),
) {}
```

The adapter binds D1 to the Worker and runs Better Auth’s schema migrations at deploy time. Git’s bucket and Durable Objects are unchanged.

## Remove the shared credential

Remove its imports from `alchemy.run.ts`:

```typescript
import * as Output from "alchemy/Output";
import * as Redacted from "effect/Redacted";
import { GitSecret } from "./src/secret.ts";
```

Return only the URL again:

```typescript
const host = yield* GitHost;
const secret = yield* GitSecret;
return {
  url: host.url.as<string>(),
  secret: Output.map(secret.text, Redacted.value),
};
return { url: host.url.as<string>() };
```

`src/secret.ts` is now unused and can be deleted. The old credential will no longer authorize requests after the next deploy.

## Deploy accounts and credentials

```sh
bun alchemy deploy
```

Install `jq` for the following shell checks if you do not already have it. Continue using the same `$HOST`.
