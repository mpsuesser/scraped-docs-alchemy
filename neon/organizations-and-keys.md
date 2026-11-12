---
url: https://alchemy.run/neon/organizations-and-keys
title: "Organizations and keys"
description: "How a Neon organization holds your projects and their branches, how an API key and orgId decide where Alchemy deploys, and why apps connect with their own credentials."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything on Neon belongs to an **organization** or to your personal account: its projects, each project’s branches, its members, and its bill. Two kinds of credential are kept apart. A **management API key** lets Alchemy create and change projects. A branch’s own credentials let your app reach its database.

acmeorganizationorganization key

my-appproject · orgId: org-…project-restricted key

mainbranch

pr-42branch

reportsproject

mainbranch

An organization holds projects, and a project holds branches. An organization key reaches every project in it, while a project-restricted key reaches only one.

Alchemy deploys with a management API key from the [Neon Console](https://console.neon.tech/). Store it in a [profile](../environments/profiles.md):

```sh
alchemy profile edit --add Neon
```

See [Setup](setup.md) for which kind of key to create.

## orgId decides the organization

A personal key acts with your own access, and creates projects in your personal account. To create one in an organization, pass its id, from **Organization Settings → General information**:

```typescript
const project = yield* Neon.Project("Application", { orgId: "org-example" });
```

`orgId` can’t change once the project exists. Choose it before you store data.

## One profile per organization

A [stage](../environments/stages.md) decides what you deploy. A profile decides where. If production lives in its own organization, give it its own key and profile:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Neon
alchemy deploy --stage prod --profile prod
```

Within one project, stages get their own [branches](data/branching.md), so a pull request never touches production’s data.

## Apps don’t hold the API key

The management key changes projects. An app connects to one branch’s database, with a [role](data/roles.md) and a [connection string](data/connections.md) for that branch. Functions and storage get branch credentials from Neon. Never put the management key in an app’s environment.

## In CI

With `CI=true`, Alchemy skips the profile and reads `NEON_API_KEY` from the environment. See [CI](../environments/ci.md).
