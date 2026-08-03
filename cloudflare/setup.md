---
url: https://alchemy.run/cloudflare/setup
title: "Setup"
description: "Install Alchemy, create a Cloudflare account, and connect the two — OAuth or API token, saved to a local profile. No environment variables required."
access_date: 2026-08-03T19:43:15.086Z
current_date: 2026-08-03T19:43:15.086Z
---

Everything you need before deploying to Cloudflare: the alchemy package, a Cloudflare account, and stored credentials.

## Install

Install `alchemy@next` and `effect@>=4.0.0-beta.102 || >=4.0.0` in your project:

```sh
bun add "alchemy@next" "effect@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-bun@>=4.0.0-beta.102 || >=4.0.0" "@effect/platform-node@>=4.0.0-beta.102 || >=4.0.0"
```

## Create a Cloudflare account

If you don’t have one yet, [sign up for Cloudflare](https://dash.cloudflare.com/sign-up) — the free plan is enough for everything in the tutorial.

## Connect alchemy to Cloudflare

There is no separate credentials step. The first time you run `alchemy deploy` (or `plan`, `dev`, `destroy`), alchemy walks each provider registered in your stack through an interactive login. For Cloudflare you can:

- **Sign in with OAuth** — opens the browser, no tokens to manage.
- **Paste an API token** — if you prefer explicit, scoped tokens.

The OAuth flow asks whether you want to customize scopes — answer yes to pick exactly which ones to grant at a multiselect prompt (the default set covers typical use).

Either way the result is saved to your **`default`** [profile](../environments/profiles.md) at `~/.alchemy/profiles.json` — no environment variables and no `wrangler login` required.

To run the flow explicitly, or re-run it later:

```sh
# Refresh credentials for the current profile
alchemy login

# Re-run the interactive setup (e.g. switch from OAuth → API token)
alchemy login --configure
```

`login` imports your stack file to discover which providers are needed, so the prompts match the providers you actually use.

## Profiles

A profile is a named bundle of credentials. Every command uses the profile named `default` unless you pass `--profile <name>` or set `$ALCHEMY_PROFILE` — handy for separating work and personal accounts, or staging and prod credentials:

```sh
# Log into a separate profile
alchemy login --profile prod --configure

# Deploy with it
alchemy deploy --stage prod --profile prod
```

Inspect what’s stored (credentials are redacted) with:

```sh
alchemy profile show
```

See [Profiles](../environments/profiles.md) for the full picture.

## Where next

- [Tutorial part 1](tutorial/part-1.md) — create a Stack, deploy your first resource.
- [Cloudflare overview](../cloudflare.md) — the map of resources and guides.
- [Secrets & env](security/secrets-env.md) — wire `.env` values and secrets into your Workers (deploy credentials live in your profile; app secrets are bindings).
