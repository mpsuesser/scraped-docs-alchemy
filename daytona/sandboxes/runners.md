---
url: https://alchemy.run/daytona/sandboxes/runners
title: "Regions and runners"
description: "Where Daytona sandboxes run — shared regions, the region a sandbox picks, and a region of your own with runners on machines you operate."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **region** is where sandboxes run, such as `us` or `eu`. Inside a region, **runners** are the machines that host sandboxes, and Daytona’s scheduler places each new sandbox on one. Daytona runs shared regions for every organization. An organization allowed to bring its own compute can add a **custom region**: Daytona still runs the API and the scheduler, and the runners are machines you operate.

## Runners host sandboxes inside a region

Homecustom region

Box1runner

user-samsandbox

user-anasandbox

Box2runner · draining

ci-417sandbox · moving off

A custom region has its own proxy and SSH gateway, and runners you operate. The scheduler places sandboxes on runners; a draining runner takes no new ones and its sandboxes move off.

## Pick a region

A sandbox runs in its organization’s default region unless it sets `target`. Changing `target` replaces the sandbox.

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";

export const DevBox = Daytona.Sandbox("DevBox", { snapshot: Base, target: "eu" });
```

Set the organization’s default with `defaultRegionId` on [OrganizationSettings](../organization/organization.md#change-organization-settings).

## Add a region of your own

```typescript
import * as Daytona from "@alchemy.run/daytona";

export const Home = Daytona.Region("Home", {
  proxyUrl: "https://proxy.daytona.example.com",
  sshGatewayUrl: "ssh.daytona.example.com:2222",
});
```

Daytona shows the region’s proxy and SSH gateway keys only when it creates or rotates them, so Alchemy keeps them in state. Bump a counter in `rotate`, like `rotate: { proxyApiKey: 1 }`, to rotate one.

```typescript
export const Box1 = Daytona.Runner("Box1", { region: Home });
```

`Box1.apiKey` is the key you start the runner daemon with on your machine.

## Drain a runner

```typescript
export const Box2 = Daytona.Runner("Box2", { region: Home });
export const Box2 = Daytona.Runner("Box2", { region: Home, draining: true });
```

A draining runner takes no new sandboxes, and its sandboxes move off. `unschedulable: true` only stops new ones. Both change in place.
