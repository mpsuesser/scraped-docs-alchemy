---
url: https://alchemy.run/axiom/guides/alerting
title: "Alerting: monitors & notifiers"
description: "Turn APL queries into alerts — Threshold, MatchEvent, and AnomalyDetection monitors wired to Slack, email, PagerDuty, and webhook notifiers, all as resources in your Stack."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

An Axiom `Monitor` is a saved APL query that runs on a fixed cadence and fires when its condition is met. A `Notifier` is the destination it fires to — Slack, email, PagerDuty, a webhook. In alchemy both are resources in your Stack, so a threshold change or a new on-call channel is a reviewable diff, not a console edit.

Everything below runs inside your Stack’s `Effect.gen` — see [Setup](../setup.md) for credentials and the [Axiom overview](../../axiom.md) for datasets like `app-traces` and `app-logs` used in the queries.

## Create a notifier

```typescript
import * as Axiom from "@alchemy.run/axiom";

const slack = yield* Axiom.Notifier("ops-slack", {
  name: "ops-channel",
  properties: {
    slack: { slackUrl: process.env.SLACK_WEBHOOK_URL! },
  },
});
```

Exactly one channel goes under `properties` — here `slack` with an incoming webhook URL. The resource’s `id` output is what monitors target.

## Threshold: alert when a value crosses a line

```typescript
yield* Axiom.Monitor("error-rate", {
  name: "High error rate",
  description: "Fires when error count exceeds 100/5m",
  type: "Threshold",
  aplQuery: \`
    ['app-traces']
    | where status >= 500
    | summarize count() by bin_auto(_time)
  \`,
  operator: "Above",
  threshold: 100,
  intervalMinutes: 5,
  rangeMinutes: 5,
  alertOnNoData: false,
  resolvable: true,
  notifierIds: [slack.id],
});
```

Every `intervalMinutes`, the monitor runs `aplQuery` over the last `rangeMinutes` and compares the aggregate against `threshold` using `operator` (`Above`, `AboveOrEqual`, `Below`, `BelowOrEqual`, or `AboveOrBelow`). `resolvable: true` sends a resolution notice when the value drops back under the line; `alertOnNoData` controls whether an empty result counts as a problem.

`notifierIds: [slack.id]` is the wiring: `slack.id` is an output attribute of the notifier flowing into the monitor’s inputs, so alchemy deploys the notifier first and fills in the real id.

<svg viewBox="0 0 692 124" style="max-width: 796px" role="img" aria-label="Each run queries the last 5 minutes and compares the result with the threshold. The first run above 100 triggers the monitor, and its notifier posts to Slack."><line stroke-opacity="0.2" stroke="currentColor" x1="355.20000000000005" y1="24" x2="355.20000000000005" y2="118"></line><g><text fill="currentColor" x="4" y="50">error-rate</text> <text fill="currentColor" x="4" y="64">every 5 minutes</text> <g><rect stroke="currentColor" fill="none" x="131.2" y="40" width="50.400000000000034" height="24" rx="5"></rect><text fill="currentColor" x="156.4" y="56" text-anchor="middle">42</text></g> <g><rect stroke="currentColor" fill="none" x="243.20000000000002" y="40" width="50.400000000000006" height="24" rx="5"></rect><text fill="currentColor" x="268.40000000000003" y="56" text-anchor="middle">87</text></g> <g><rect stroke="currentColor" fill="none" x="355.20000000000005" y="40" width="50.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="380.40000000000003" y="56" text-anchor="middle">131</text></g> <g><rect stroke="currentColor" fill="none" x="467.2" y="40" width="50.39999999999992" height="24" rx="5"></rect><text fill="currentColor" x="492.4" y="56" text-anchor="middle">156</text></g> <g><rect stroke="currentColor" fill="none" x="579.2" y="40" width="50.399999999999864" height="24" rx="5"></rect><text fill="currentColor" x="604.4" y="56" text-anchor="middle">64</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="692" y2="74"></line><text fill="currentColor" x="4" y="94">notifier</text> <text fill="currentColor" x="4" y="108">Slack</text> <g><rect stroke="currentColor" fill="none" x="366.40000000000003" y="84" width="56" height="24" rx="5"></rect><text fill="currentColor" x="428.40000000000003" y="100" text-anchor="start">triggered</text></g></g> <text fill="currentColor" x="355.20000000000005" y="18" text-anchor="middle">above 100</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="Each run queries the last 5 minutes and compares the result with the threshold. The first run above 100 triggers the monitor, and its notifier posts to Slack."><line stroke-opacity="0.2" stroke="currentColor" x1="133.56" y1="24" x2="133.56" y2="138"></line><g><text fill="currentColor" x="0" y="44">error-rate <tspan fill="currentColor" dx="8">every 5 minutes</tspan></text> <g><rect stroke="currentColor" fill="none" x="6.36" y="54" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="20.67" y="70" text-anchor="middle">42</text></g> <g><rect stroke="currentColor" fill="none" x="69.96000000000001" y="54" width="28.61999999999999" height="24" rx="5"></rect><text fill="currentColor" x="84.27000000000001" y="70" text-anchor="middle">87</text></g> <g><rect stroke="currentColor" fill="none" x="133.56" y="54" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="168.18" y="70" text-anchor="start">131</text></g> <g><rect stroke="currentColor" fill="none" x="197.16" y="54" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="231.78" y="70" text-anchor="start">156</text></g> <g><rect stroke="currentColor" fill="none" x="260.76" y="54" width="28.620000000000005" height="24" rx="5"></rect><text fill="currentColor" x="275.07" y="70" text-anchor="middle">64</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">notifier <tspan fill="currentColor" dx="8">Slack</tspan></text> <g><rect stroke="currentColor" fill="none" x="139.92000000000002" y="108" width="31.799999999999983" height="24" rx="5"></rect><text fill="currentColor" x="177.72" y="124" text-anchor="start">triggered</text></g></g> <text fill="currentColor" x="133.56" y="18" text-anchor="middle">above 100</text></svg>

Each run queries the last 5 minutes and compares the result with the threshold. The first run above 100 triggers the monitor, and its notifier posts to Slack.

## MatchEvent: alert on every matching event

```typescript
yield* Axiom.Monitor("panics", {
  name: "Service panic",
  type: "MatchEvent",
  aplQuery: \`['app-logs'] | where message contains "panic:"\`,
  intervalMinutes: 1,
  rangeMinutes: 1,
  notifierIds: [slack.id],
});
```

A `MatchEvent` monitor has no threshold — it fires for every event the query matches. Use it for rare, individually-actionable events like panics or failed deploys, where one occurrence is already worth a page.

## AnomalyDetection: alert on deviation from baseline

```typescript
yield* Axiom.Monitor("traffic-anomaly", {
  name: "Traffic anomaly",
  type: "AnomalyDetection",
  aplQuery: \`['app-traces'] | summarize count() by bin_auto(_time)\`,
  compareDays: 7,
  tolerance: 25, // %
  intervalMinutes: 15,
  rangeMinutes: 15,
  notifierIds: [slack.id],
});
```

Instead of a static threshold, the monitor learns a baseline from the same window over the previous `compareDays` days and fires when the current result deviates by more than `tolerance` percent. Good for “traffic looks wrong” signals where the right absolute number changes by hour and weekday.

Changing a monitor’s `type` triggers a replacement; every other prop updates in place.

## Fan out to multiple notifiers

```typescript
const pagerduty = yield* Axiom.Notifier("pagerduty", {
  name: "primary-oncall",
  properties: {
    pagerduty: { routingKey: process.env.PAGERDUTY_ROUTING_KEY!, token: "" },
  },
});

yield* Axiom.Monitor("error-rate", {
  // ...same props as above
  notifierIds: [slack.id, pagerduty.id],
});
```

`notifierIds` takes any number of ids, so one monitor can post to the team channel and page on-call at the same time. Notifier channels cover Slack, email (`email: { emails: [...] }`), PagerDuty, Opsgenie, Discord, Microsoft Teams, and generic webhooks.

## Custom webhook with a templated body

```typescript
yield* Axiom.Notifier("incident-webhook", {
  name: "incident.io",
  properties: {
    customWebhook: {
      url: "https://api.incident.io/v2/alert_events",
      headers: { "Content-Type": "application/json" },
      secretHeaders: { Authorization: \`Bearer ${process.env.INCIDENT_TOKEN}\` },
      body: '{"title": "{{.Monitor.Name}}", "status": "firing"}',
    },
  },
});
```

For anything without a built-in channel, `customWebhook` gives you full control over the request: static `headers`, `secretHeaders` for credentials, and a `body` template with access to monitor fields like `{{.Monitor.Name}}`.

## Derived alert fields with VirtualField

```typescript
yield* Axiom.VirtualField("status-class", {
  dataset: "app-traces",
  name: "status_class",
  description: "HTTP response class bucket",
  expression: 'strcat(tostring(toint(status / 100)), "xx")',
  type: "string",
});
```

A `VirtualField` is a saved APL expression that appears as a derived column on its `dataset` at query time. Define common computations — status classes, latency buckets, parsed JSON paths — once, instead of repeating the expression in every monitor and dashboard. A virtual field is bound to one `dataset`; changing the dataset triggers a replacement.

Monitors on that dataset can then query the derived column directly:

```typescript
yield* Axiom.Monitor("5xx-rate", {
  name: "5xx rate",
  type: "Threshold",
  aplQuery: \`
    ['app-traces']
    | where status_class == "5xx"
    | summarize count() by bin_auto(_time)
  \`,
  operator: "Above",
  threshold: 50,
  intervalMinutes: 5,
  rangeMinutes: 5,
  notifierIds: [slack.id],
});
```

If the definition of “5xx” ever changes, you update the virtual field’s `expression` — every monitor and dashboard built on `status_class` follows.

## Where next

- [Ingest](../data/ingest.md) — datasets, ingest tokens, and wiring OTLP endpoints into your Workers and Functions.
- [Dashboards](dashboards.md) — chart the same queries your monitors run.
- [Axiom overview](../../axiom.md) — all Axiom resources at a glance.

Reference:

- [Monitor API reference](https://alchemy.run/providers/axiom/reference/monitor#monitor)
- [Notifier API reference](https://alchemy.run/providers/axiom/reference/monitor#notifier)
- [VirtualField API reference](https://alchemy.run/providers/axiom/reference/dataset#virtualfield)
