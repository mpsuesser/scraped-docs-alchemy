---
url: https://alchemy.run/github/accounts-and-tokens
title: "Accounts and tokens"
description: "How GitHub accounts and organizations own repositories, which token Alchemy acts as and the scopes it needs, and how OIDC lets workflows reach a cloud with no stored secret."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A repository belongs to an **account**: a person, or an **organization** that many people share, with teams and per-repository permissions. Alchemy acts as whoever its GitHub token belongs to, so the token decides which repositories it can change.

acmeorganization · teams, permissionstoken: repo, workflow

legacyrepository · not managed

apirepository

webrepository

Alchemy acts as whoever owns its token, so the token's scopes and that account's access decide which repositories it can change. A machine account can be limited to the repositories it manages.

Add GitHub to your [profile](../environments/profiles.md):

```sh
alchemy profile edit --add GitHub
```

The simplest choice is the **`gh` CLI**: Alchemy asks `gh auth token` for your current login, so there’s nothing to copy. Or store a **personal access token**. See [Setup](setup.md).

## Token scopes

A token needs `repo` to manage repositories, and `workflow` to set [Actions secrets and variables](actions-config.md). It needs `delete_repo` only if you let Alchemy delete a repository, since repositories are kept by default when they leave your Stack.

Use a machine account or a narrow token for automation, so it reaches only the repositories it manages.

## Secrets and variables flow from your Stack

A workflow reads credentials from **secrets** and settings from **variables**. Alchemy sets them from values your Stack produces, so a credential goes from the cloud to CI without anyone copying it. See [Actions secrets & variables](actions-config.md).

## OIDC: no stored secret at all

Better than a secret is no secret. A workflow can prove which repository and branch it’s running for with an **OIDC token** GitHub signs, and a cloud that trusts GitHub, like an AWS role with GitHub as its identity provider, hands it short-lived credentials. The [AWS CI tutorial](../aws/tutorial/part-5.md) sets this up as code.

## In CI

With `CI=true`, Alchemy skips the profile and reads `GITHUB_ACCESS_TOKEN` or `GITHUB_TOKEN` from the environment. See [CI](../environments/ci.md).
