---
url: https://alchemy.run/hetzner/setup
title: "Setup"
description: "Install Alchemy and connect it to Hetzner Cloud — create a project, generate an API token, and store it in a profile."
access_date: 2026-09-09T22:57:45.923Z
current_date: 2026-09-09T22:57:45.923Z
---

Everything you need before deploying to Hetzner Cloud: the alchemy package, a Hetzner project, and an API token stored in a profile.

## Prerequisites

- [Bun](https://bun.sh/) (recommended) or Node.js 22+
- A [Hetzner Cloud](https://console.hetzner.com/) account

## Create a Hetzner project

Everything in Hetzner Cloud — servers, volumes, networks, tokens — lives inside a **project**. In the [Hetzner Cloud Console](https://console.hetzner.com/):

1. Click **\+ New project**.
2. Give it a name (e.g. `my-app`) and open it.

One project per app (or per environment) is a good default: API tokens are scoped to a single project, so separate projects give you hard isolation between apps.

## Generate an API token

Inside the project:

1. Go to **Security** → **API tokens**.
2. Click **Generate API token**.
3. Give it a description (e.g. `alchemy`) and select **Read & Write** — Alchemy needs write access to create resources.
4. Copy the token — Hetzner shows it **only once**.

## Create a project directory

```sh
mkdir my-app && cd my-app && bun init -y
```

## Install

```sh
bun add "alchemy@latest" "effect@rc" "@effect/platform-bun@rc" "@effect/platform-node@rc"
```

## Connect alchemy to Hetzner

There is no separate credentials step. The first time you run `alchemy deploy` (or `plan`, `dev`, `destroy`) on a stack that uses `Hetzner.providers()`, alchemy prompts for the token you generated above (and, optionally, an API endpoint override). It is saved under `~/.alchemy/credentials/<profile>/` in the selected [profile](../environments/profiles.md) and reused on subsequent commands.

In CI (`CI=true`) alchemy skips the prompt and reads `HCLOUD_TOKEN` (plus an optional `HCLOUD_ENDPOINT`) from the environment instead.

To re-run the setup later (e.g. to rotate the token, or configure a separate `prod` profile):

```sh
# Re-run the interactive setup (e.g. to rotate the token)
alchemy profile edit --reconfigure Hetzner

# Or connect Hetzner in a separate \`prod\` profile
alchemy profile create prod
alchemy profile edit --profile prod --add Hetzner

# Non-interactive (scripts, agents)
alchemy profile edit --add Hetzner --method stored --set token=env:HCLOUD_TOKEN
```

Inspect what’s stored (secrets are redacted):

```sh
alchemy profile show
```

## State storage

Hetzner has no object-storage state backend of its own, so pick one of:

- **`Alchemy.localState()`** — state on disk under `.alchemy/` next to your code. Zero setup; right for solo projects and trying things out.
- **A cloud state store** — if you also use Cloudflare or AWS, pass `Cloudflare.state()` or `AWS.state()` so state is shared with your team and CI. See [State Store](../state-store.md).

State for a Hetzner stack includes each Server’s deploy SSH key (stored redacted), so for shared or production stacks prefer a remote state store over files on one laptop.

## Next steps

- [Tutorial part 1](https://alchemy.run/hetzner/tutorial/part-1) — deploy your first Server.
- [Hetzner overview](../hetzner.md) — the map of resources and guides.
