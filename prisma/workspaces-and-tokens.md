---
url: https://alchemy.run/prisma/workspaces-and-tokens
title: "Workspaces and tokens"
description: "How a Prisma workspace holds your projects and billing, how a service token decides where Alchemy deploys, and how apps get their own database connections."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything on Prisma’s data platform belongs to a **workspace**: its projects, with their Postgres databases and Compute apps, its members, and its bill.

acmeworkspace · members, billservice token: deploys

reportsproject

reportsPostgres

my-appproject

appPostgres

apiCompute app

The service token reaches the whole workspace and stays with Alchemy. An app reaches only its own database, with a connection string made for it.

## A service token decides the workspace

Alchemy deploys with a **service token**, created in the [Prisma Console](https://console.prisma.io/) under workspace settings. A service token belongs to the workspace, not to a person, and Alchemy deploys into the workspace it belongs to. Store it in a [profile](../environments/profiles.md):

```sh
alchemy profile edit --add Prisma
```

See [Setup](setup.md).

## One workspace per environment

A [stage](../environments/stages.md) decides what you deploy. A profile decides where. To keep production’s members and bill apart, give it its own workspace, token and profile:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Prisma
alchemy deploy --stage prod --profile prod
```

## Apps don’t hold the service token

The service token manages the workspace. An app only connects to its database, with a connection string that reaches that one database. See [Connections](data/connections.md).

## In CI

With `CI=true`, Alchemy skips the profile and reads `PRISMA_SERVICE_TOKEN` from the environment. See [CI](../environments/ci.md).
