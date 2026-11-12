---
url: https://alchemy.run/axiom
title: "Axiom"
description: "Observability as resources — OTEL datasets, ingest tokens, monitors, notifiers, and dashboards declared next to the code that emits the data."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Axiom is a platform for event data and observability. It stores logs, traces and metrics at any scale, and queries them in seconds.

You send it OpenTelemetry and query it with APL. Monitors alert you when something crosses a line, and dashboards show what’s happening.

**New to Axiom?** Start with [What is Axiom?](axiom/what-is-axiom.md).

## A quick example

A Dataset holds one kind of telemetry, and exposes its OTLP endpoint:

```typescript
const traces = yield* Axiom.Dataset("traces", { name: "app-traces", kind: "otel:traces:v1" });
const logs = yield* Axiom.Dataset("logs", { name: "app-logs", kind: "otel:logs:v1" });
```

An ApiToken gives your runtime ingest access to one dataset, and nothing else:

```typescript
const ingest = yield* Axiom.ApiToken("ingest", {
  name: "prod-ingest",
  datasetCapabilities: {
    "app-traces": { ingest: ["create"] },
  },
});
```

A Monitor runs an APL query on a schedule and alerts through a Notifier:

```typescript
const slack = yield* Axiom.Notifier("ops-slack", {
  name: "ops-channel",
  properties: {
    slack: { slackUrl: yield* Config.String("SLACK_WEBHOOK_URL") },
  },
});

yield* Axiom.Monitor("panics", {
  name: "Service panic",
  type: "MatchEvent",
  aplQuery: \`['app-logs'] | where message contains "panic:"\`,
  intervalMinutes: 1,
  rangeMinutes: 1,
  notifierIds: [slack.id],
});
```

Point a Worker or Lambda’s OpenTelemetry exporter at the dataset endpoints, and Effect’s traces and logs flow into Axiom with no code changes. See [Observability](testing/observability.md).

## Where to start

**Want to build?** Connect your organization in [Setup](axiom/setup.md), then [Observability with Axiom](cloudflare/observability/axiom-observability.md) takes a Worker from no telemetry to a monitor that pages you.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Accounts & access

- [Organizations and tokens](axiom/organizations-and-tokens.md)

### Data

- [Datasets & ingest](axiom/data/ingest.md)

### Guides

- [Alerting](axiom/guides/alerting.md)
- [Dashboards](axiom/guides/dashboards.md)
- [Annotations](axiom/guides/annotations.md)
