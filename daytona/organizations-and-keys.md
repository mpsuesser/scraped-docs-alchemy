---
url: https://alchemy.run/daytona/organizations-and-keys
title: "Organizations and API keys"
description: "How a Daytona organization holds your sandboxes and bill, how your profile picks the organization Alchemy deploys to, and the API key each Worker gets."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything you run on Daytona belongs to an **organization**: its sandboxes, snapshots, volumes and secrets, the API keys that may touch them, its members, and its bill. Nothing in one organization can see another’s, so an organization is also a boundary.

## A key’s permissions cover a kind of object

Every call to Daytona carries an **API key**, and every key belongs to exactly one organization. A key’s permissions apply to a kind of object, not to one object: a key with `write:sandboxes` can create any sandbox in its organization.

acmeorganization · one billkey reaches every sandbox here

ApiBaseSandboxesApiKeyAPI key

permissions

write:sandboxes · delete:sandboxes

user-samsandbox

user-anasandbox

Basesnapshot

An API key belongs to one organization, and its permissions cover every object of a kind in it. The Api Worker's key can create and delete any sandbox in acme, and nothing else.

Alchemy reads your Daytona credentials from a [profile](../environments/profiles.md):

```sh
alchemy profile edit --add Daytona
```

Pick **Sign in** for most setups. Alchemy opens Daytona’s device login, the same one `daytona login` uses, and the session refreshes itself. Signing in is what lets Alchemy create an API key for each Worker that uses a binding: Daytona only lets a signed-in user create keys, organizations, roles and invitations.

Pick **API key** to paste a key from **Keys** in the dashboard instead. A key covers sandboxes, snapshots, volumes, secrets and registries, but it can create the per-Worker keys bindings need only if Daytona has enabled managed API keys for your organization and the key has `manage:api_keys`.

See [Setup](setup.md) for the details.

## The profile picks the organization

With **Sign in**, Alchemy lists your organizations and asks which one to use if you belong to several. The profile stores that choice, and every deploy with the profile goes there. With an **API key**, the deploy goes to the key’s organization.

A sandbox runs in the organization’s default region unless it sets `target`. See [Regions and runners](sandboxes/runners.md).

## Give production its own organization

A [stage](../environments/stages.md) decides what you deploy. The profile decides where. Give production its own organization, so its sandboxes, keys, members and bill are separate, and a profile of its own:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Daytona
alchemy deploy --stage prod --profile prod
```

Many stages can share a development organization, each with its own snapshots and sandboxes. A Stack deployed with a signed-in profile can also create an organization:

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Team = Daytona.Organization("Team", { defaultRegionId: "us" });
```

## Each Worker gets a key of its own

Each binding creates its own key for its Worker or Lambda, named after the host, with only what its calls need:

| Binding | Permissions |
| --- | --- |
| `CreateSandbox`, `ConnectSandbox` | `write:sandboxes`, `delete:sandboxes` |
| `CreateVolume` | `read:volumes`, `write:volumes`, `delete:volumes` |
| `CreateSecret` | `manage:secrets` |
| `CreateSnapshot` | `write:snapshots`, `delete:snapshots`, `write:sandboxes`, `delete:sandboxes` |
| `ReadOrganization` | `read:limits` |
| `ReadUsage` | `read:billing` |
| `ReadAuditLogs` | `read:audit_logs` |

Because permissions apply per kind, the bindings also label what they create and stay inside it: `CreateSandbox(Base)` only lists sandboxes created from `Base`, and `CreateVolume({ prefix: "user-" })` refuses volume names that don’t start with `user-`.

## Declare a key for CI or another app

For CI or a program outside your Stack, declare a key:

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Ci = Daytona.ApiKey("Ci", {
  permissions: ["write:snapshots", "delete:snapshots"],
});
```

`Ci.value` is the key. Daytona shows it once, and Alchemy keeps it in state. Changing the permissions creates a new key and deletes the old one, and destroying it deletes the key.

## Deploy from CI with a key

With `CI=true`, Alchemy skips the profile and reads `DAYTONA_API_KEY`. A Stack with bindings needs a key with `manage:api_keys`. The deploy goes to the key’s organization, or to `DAYTONA_ORGANIZATION_ID` when you set it. Set `DAYTONA_API_URL` only for a self-hosted Daytona. See [CI](../environments/ci.md).

Members, roles and single sign-on are in [Members and roles](organization/organization.md). Values a sandbox uses but can’t read are in [Secrets](security/secrets.md).
