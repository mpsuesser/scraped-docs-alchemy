---
url: https://alchemy.run/turso/organization
title: "Team & organization"
description: "Invite teammates and change your Turso organization's settings."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

These work on the organization your [profile](setup.md) names, or your token’s organization when it names none.

Invites and members need a team organization on a paid plan. The free plan doesn’t allow them.

acmeorganization

anamember · admin

sammember · viewer

Tenantsgroup

acmedatabase

globexdatabase

Analyticsgroup

eventsdatabase

An organization holds the groups and their databases, and the members who can see or change them. Your profile or token decides which organization Alchemy deploys into.

## Invite a teammate

```typescript
yield* Turso.OrganizationInvite("Alice", {
  email: "alice@example.com",
  role: "admin",
});
```

`role` is `admin`, `member`, or `viewer`. The default is `member`. Changing it sends the invite again. Destroying it cancels the invite.

## Add an existing user

```typescript
yield* Turso.OrganizationMember("Bob", {
  username: "bob",
  role: "viewer",
});
```

Use this when the person already has a Turso account. You can change `role` later. Destroying it removes them.

## Change organization settings

```typescript
yield* Turso.OrganizationSettings("Org", {
  requireMfa: true,
  restoreEnabled: true,
});
```

Alchemy only touches the settings you list. Destroying it puts them back the way they were.

`overages` lets Turso bill you for usage past your plan’s limits. Only turn it on if you mean to.

## Where next

- [Organization reference](https://alchemy.run/providers/turso/reference/organization)
