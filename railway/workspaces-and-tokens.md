---
url: https://alchemy.run/railway/workspaces-and-tokens
title: "Workspaces and tokens"
description: "How a Railway workspace holds your projects and billing, which tokens Alchemy can deploy with, and how to keep production apart."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything you run on Railway belongs to a **workspace**: your personal account, or a team you share with others. A workspace holds your projects, their members and their bill. Inside it, each **project** groups services and databases on their own private network.

acmeworkspace · members, billworkspace or account token

my-app-prodproject · one per stage

apiService

Postgresdatabase

my-app-pr-42project

apiService

Postgresdatabase

A workspace holds a project per stage, and each project its services and databases on a private network. Alchemy needs a token that reaches the workspace, because it creates the projects.

Alchemy reaches a workspace through a [profile](../environments/profiles.md):

```sh
alchemy profile edit --add Railway
```

**OAuth** pairs Alchemy with your Railway account in the browser, with no token to create. Or paste an **account** or **workspace token**, made under **Account Settings → Tokens**. A **project token** won’t work: it reaches one project, and Alchemy creates projects.

A Project is created in your token’s workspace, or, for an account token or OAuth, in your first workspace. To pin it to another workspace your token can reach, set `workspaceId`:

```typescript
export const Site = Railway.Project("Site", { workspaceId: "ws_123" });
```

See [Setup](setup.md) for the details.

## Keep production apart

A [stage](../environments/stages.md) decides what you deploy, and each stage gets its own project. For a stronger line around production, give it its own workspace and profile, so its members and its bill are separate:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Railway
alchemy deploy --stage prod --profile prod
```

## In CI

With `CI=true`, Alchemy skips the profile and reads `RAILWAY_API_TOKEN` from the environment. See [CI](../environments/ci.md).

## Next

Values your Services read at run time are covered in [Variables](data/variables.md).
