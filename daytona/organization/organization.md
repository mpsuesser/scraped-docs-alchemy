---
url: https://alchemy.run/daytona/organization/organization
title: "Members and roles"
description: "Give people access to a Daytona organization with roles, invitations and members, sign them in with single sign-on, and set organization defaults."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

People reach an organization as **members**. Each member is an `owner` or a `member`, and a member also gets **roles**: named sets of the same permissions API keys carry. Someone who isn’t in the organization yet gets an **invitation**, with the roles they’ll have once they accept.

## A role is a set of permissions

acmeorganization

Developerrole

bob@example.commember

alice@example.cominvitation · pending

A role is a set of permissions. Bob is a member with the Developer role; Alice gets the same role when she accepts her invitation.

## Create a role

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Developer = Daytona.OrganizationRole("Developer", {
  description: "Run sandboxes",
  permissions: ["write:sandboxes", "delete:sandboxes", "read:volumes"],
});
```

## Invite someone

```typescript
export const Alice = Daytona.OrganizationInvitation("Alice", {
  email: "alice@example.com",
  roles: [Developer],
});
```

Destroy cancels the invitation if it is still pending.

## Change a member’s access

```typescript
export const Bob = Daytona.OrganizationMember("Bob", {
  user: "bob@example.com",
  role: "member",
  roles: [Developer],
});
```

Destroy puts back the access Bob had before. Set `removeOnDestroy: true` to remove him instead.

## Sign members in with SSO

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Config from "effect/Config";
import { Developer } from "./Team.ts";

export const Sso = Daytona.IdentityProvider("Sso", {
  issuerUrl: "https://login.example.com",
  clientId: "daytona",
  clientSecret: Config.Redacted("OIDC_CLIENT_SECRET"),
  emailDomains: ["example.com"],
  defaultRoles: [Developer],
});
```

People with an address in `emailDomains` sign in through your identity provider, and new members get the roles in `defaultRoles`. SSO needs a Daytona plan that includes it.

## Change organization settings

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Settings = Daytona.OrganizationSettings("Settings", {
  defaultRegionId: "eu",
  defaultQueueTimeout: "1 minute",
});
```

Only the settings you declare change, and destroy puts the old values back.

## Some actions stay in the dashboard

Signing in, accepting invitations and linking accounts act as a person, not as your organization, so you do them in the Daytona dashboard.
