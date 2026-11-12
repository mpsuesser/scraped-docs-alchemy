---
url: https://alchemy.run/cloudflare/accounts-and-tokens
title: "Accounts and API tokens"
description: "How a Cloudflare account holds your Workers, storage and zones, how Alchemy signs in to it with OAuth or an API token, and how your Stack mints scoped tokens for CI and runtimes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything you deploy to Cloudflare lives in an **account**: Workers, KV namespaces, R2 buckets, D1 databases, queues and your zones. An account has **members**, the people who can sign in to it, each with roles that decide what they can change.

my-companyaccount · members: ana, sam

ApiWorker

SessionsKV namespace

UploadsR2 bucket

example.comzonetoken: DNS Write

apiDNS record

wwwDNS record

An account holds your zones and everything you deploy. A token's policy can reach the whole account, or one zone, like this one that may edit only example.com's DNS.

Alchemy reaches an account through a [profile](../environments/profiles.md). Add Cloudflare to your `default` profile:

```sh
alchemy profile edit --add Cloudflare
```

**OAuth** is the simplest choice: it opens your browser, you approve the scopes, and Alchemy refreshes the session for you. On the consent page, turn on **Account API Token Provisioning**, so Alchemy can mint scoped tokens for your Workers. If you’d rather manage credentials yourself, paste an **API token** instead.

If you belong to several accounts, the profile records which one to deploy into. See [Setup](setup.md).

## Use one profile per account

A [stage](../environments/stages.md) decides what you deploy. A profile decides where. Keep production in its own account, with its own profile:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Cloudflare
alchemy deploy --stage prod --profile prod
```

Many stages can share the development account, each with its own Workers and storage.

## API tokens: access narrower than yours

An **API token** carries a list of policies. Each policy grants **permission groups**, like `Workers Scripts Write`, on a set of resources: a whole account, one zone, or one Worker. A token can do only what its policies say, whoever created it.

**Account-owned tokens** belong to the account rather than to a person, so they keep working when someone leaves. Declare one in your Stack to give CI exactly what it needs:

```typescript
const { accountId } = yield* yield* Cloudflare.CloudflareEnvironment;

const token = yield* Cloudflare.ApiToken.AccountApiToken("ci-token", {
  policies: [
    {
      effect: "allow",
      permissionGroups: ["Workers Scripts Write", "Workers KV Storage Write"],
      resources: { [\`com.cloudflare.api.account.${accountId}\`]: "*" },
    },
  ],
});
```

`token.value` is an output. Pass it to whatever needs it, like a [GitHub Actions secret](../github/actions-config.md), and no one copies it by hand.

## Tokens your runtimes get for you

Most of the time a Worker reaches KV, R2 or a queue through a native **binding**, with no token at all. Code that runs elsewhere, like a Lambda Function or a Container, reaches the same resources over Cloudflare’s HTTP API. Its HTTP binding mints an account-owned token with only the permission groups that binding needs, and passes it to the runtime. That’s why OAuth asks for Account API Token Provisioning.

## In CI

CI can’t open a browser. With `CI=true`, Alchemy reads credentials from the environment instead: `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN`. See [CI](../environments/ci.md).

## Next

Values your Workers read, like a third-party API key, are covered in [Secrets & env](security/secrets-env.md). Secrets shared by several Workers belong in the [Secrets Store](security/secrets-store.md). To put sign-in in front of an app, see [Access](security/access.md).
