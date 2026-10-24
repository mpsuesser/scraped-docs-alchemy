---
url: https://alchemy.run/cloudflare/setup
title: "Setup"
description: "Install Alchemy, create a Cloudflare account, and connect the two — OAuth or API token, saved to a local profile. No environment variables required."
access_date: 2026-10-24T00:47:37.929Z
current_date: 2026-10-24T00:47:37.929Z
---

Everything you need before deploying to Cloudflare: the alchemy and `@alchemy.run/cloudflare` packages, a Cloudflare account, and stored credentials.

## Install

Install `alchemy@latest`, `@alchemy.run/cloudflare` and `effect` in your project:

```sh
pnpm add alchemy@latest @alchemy.run/cloudflare@latest effect @effect/platform-bun @effect/platform-node
```

Alchemy runs on [Bun](https://bun.sh/) or Node.js 22+.

## Create a Cloudflare account

If you don’t have one yet, [sign up for Cloudflare](https://dash.cloudflare.com/sign-up) — the free plan is enough for everything in the tutorial.

## Connect alchemy to Cloudflare

Connect Cloudflare (the `default` profile is created automatically on first use):

```sh
alchemy profile edit --add Cloudflare
```

For Cloudflare you can:

- **Sign in with OAuth** — opens the browser, no tokens to manage.
- **Paste an API token** — if you prefer explicit, scoped tokens.

The OAuth flow asks whether you want to customize scopes — answer yes to pick exactly which ones to grant at a multiselect prompt (the default set covers typical use).

On Cloudflare’s consent page, turn on **Account API Token Provisioning** under **Sensitive scopes**. It’s off by default, and alchemy needs it to mint scoped API tokens on your behalf.

![Cloudflare’s consent page with the Account API Token Provisioning toggle under Sensitive scopes](https://alchemy.run/cloudflare/oauth-api-token-provisioning.png)

Either way the result is saved to your **`default`** [profile](../environments/profiles.md) at `~/.alchemy/profiles.json` — no environment variables and no `wrangler login` required.

To re-run the flow later:

```sh
# Re-run the interactive setup (e.g. switch from OAuth → API token)
alchemy profile edit --reconfigure Cloudflare

# Or manage every account in the profile from one menu
alchemy profile edit
```

`profile edit` imports your stack file to discover which providers are available, so the menu matches the providers you actually use.

## Profiles

A profile is a named bundle of credentials. Commands use the built-in `default` profile unless you name another one explicitly. Pass `--profile <name>` or set `$ALCHEMY_PROFILE` to select one — handy for separating work and personal accounts, or staging and prod credentials:

```sh
# Log into a separate profile
alchemy profile create prod
alchemy profile edit --profile prod

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
