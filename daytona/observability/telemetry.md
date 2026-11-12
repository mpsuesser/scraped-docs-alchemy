---
url: https://alchemy.run/daytona/observability/telemetry
title: "Telemetry, webhooks and audit"
description: "Send a Daytona organization's sandbox logs, traces and metrics to your backend, get webhooks for sandbox events, and read usage and the audit log from a Worker."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Daytona records what happens in your organization in three places. Sandboxes send **telemetry** (logs, traces and metrics) to Daytona’s collector, or to your own OpenTelemetry backend. **Webhooks** deliver events about your sandboxes and snapshots to endpoints you add. And the **audit log** keeps every change made to the organization: who or what did it, the action, and the object it touched, including the stops and archives Daytona makes on its own.

## Three records of what happened

Audit log · acmeDaytona's own actions

| time | action | target |
| --- | --- | --- |
| 10:02 | create | sandbox user-sam |
| 10:05 | update\_network\_settings | sandbox user-sam |
| 10:20 | auto\_stop | sandbox user-sam |
| 11:41 | start | sandbox user-sam |
| +7 days | auto\_archive | sandbox user-sam |

## Export telemetry to your backend

```typescript
import * as Daytona from "@alchemy.run/daytona";
import * as Config from "effect/Config";

export const Telemetry = Daytona.OrganizationOtelConfig("Telemetry", {
  endpoint: "https://api.axiom.co",
  headers: { Authorization: Config.Redacted("AXIOM_TOKEN"), "X-Axiom-Dataset": "daytona" },
});
```

Every sandbox in the organization sends its logs, traces and metrics to your backend. One sandbox can send its own elsewhere with `otelEndpointOverride`; its telemetry then doesn’t show in Daytona’s dashboard.

## Read a sandbox’s logs

```typescript
const logs = yield* sandbox.getSandboxLogs({ from, to });
```

## Turn on webhooks

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Events = Daytona.Webhooks("Events", {});
```

Add endpoints in Daytona’s webhook portal. A Worker gets a link to the portal from `ReadOrganization` ’s `getWebhookControllerAppPortalAccess`.

## Read usage

```typescript
const usage = yield* Daytona.ReadUsage();
const rows = yield* usage.listOrganizationUsageSandbox({ from, to });
```

The Worker’s key has `read:billing`, and the rows break down usage per sandbox.

## Read the audit log

```typescript
const audit = yield* Daytona.ReadAuditLogs();
const page = yield* audit.getOrganizationAuditLogs({ limit: 50 });
```

Each entry has the actor (an email, or the prefix of the API key used), the action, the target and its id, and the response status. The Worker’s key has `read:audit_logs`.
