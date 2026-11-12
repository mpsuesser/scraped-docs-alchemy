---
url: https://alchemy.run/gcp/observability/monitoring
title: "Logging & Monitoring"
description: "Declare uptime checks, alert policies, notification channels, and logs-based metrics in the same Stack as the services they watch."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Cloud Run sends everything your service logs to Cloud Logging, with no setup. Declare the checks and alerts that watch it next to the service itself.

## Log from your code

`Effect.log` goes to Cloud Logging:

```typescript
yield* Effect.log(\`order ${id} confirmed\`);
yield* Effect.logWarning(\`retrying ${id}\`);
```

## Check that the service is up

An uptime check probes a URL from several regions:

```typescript
Effect.gen(function* () {
  const api = yield* Api;

  const check = yield* GCP.Monitoring.UptimeCheckConfig("ApiUp", {
    httpCheck: { path: "/", useSsl: true, validateSsl: true },
    monitoredResource: {
      type: "uptime_url",
      labels: { host: Output.map(api.uri, (uri) => new URL(uri!).host) },
    },
    period: "300s",
  });
}),
```

## Get notified

Create a channel to send alerts to:

```typescript
const oncall = yield* GCP.Monitoring.NotificationChannel("Oncall", {
  type: "email",
  labels: { email_address: "oncall@example.com" },
});
```

## Alert on a metric

An alert policy opens an incident when a condition holds. This one fires when the service answers with 5xx errors:

```typescript
yield* GCP.Monitoring.AlertPolicy("ApiErrors", {
  combiner: "OR",
  conditions: [
    {
      displayName: "5xx responses",
      conditionThreshold: {
        filter:
          'resource.type = "cloud_run_revision" AND metric.type = "run.googleapis.com/request_count" AND metric.labels.response_code_class = "5xx"',
        comparison: "COMPARISON_GT",
        thresholdValue: 5,
        duration: "60s",
        aggregations: [{ alignmentPeriod: "60s", perSeriesAligner: "ALIGN_RATE" }],
      },
    },
  ],
  notificationChannels: [oncall.name],
});
```

<svg viewBox="0 0 679.2 124" style="max-width: 781px" role="img" aria-label="The condition has to hold for its whole duration before the policy opens an incident and notifies the channel. When the errors stop, the incident closes."><line stroke-opacity="0.2" stroke="currentColor" x1="252.8" y1="24" x2="252.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="555.2" y1="24" x2="555.2" y2="118"></line><g><text fill="currentColor" x="4" y="50">5xx &gt; 5/s</text> <text fill="currentColor" x="4" y="64">the condition</text> <g><rect stroke="currentColor" fill="none" x="163.2" y="40" width="392.00000000000006" height="24" rx="5"></rect><text fill="currentColor" x="359.20000000000005" y="56" text-anchor="middle">condition holds</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="679.2" y2="74"></line><text fill="currentColor" x="4" y="94">ApiErrors</text> <text fill="currentColor" x="4" y="108">incident</text> <g><rect stroke="currentColor" fill="none" x="252.8" y="84" width="302.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="404" y="100" text-anchor="middle">open</text></g></g> <text fill="currentColor" x="252.8" y="18" text-anchor="middle">held 60s: oncall notified</text> <text fill="currentColor" x="555.2" y="18" text-anchor="middle">clears: closes</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="The condition has to hold for its whole duration before the policy opens an incident and notifies the channel. When the errors stop, the incident closes."><line stroke-opacity="0.2" stroke="currentColor" x1="82.68" y1="24" x2="82.68" y2="138"></line><line stroke-opacity="0.2" stroke="currentColor" x1="254.4" y1="24" x2="254.4" y2="138"></line><g><text fill="currentColor" x="0" y="44">5xx &gt; 5/s <tspan fill="currentColor" dx="8">the condition</tspan></text> <g><rect stroke="currentColor" fill="none" x="31.8" y="54" width="222.6" height="24" rx="5"></rect><text fill="currentColor" x="143.1" y="70" text-anchor="middle">condition holds</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">ApiErrors <tspan fill="currentColor" dx="8">incident</tspan></text> <g><rect stroke="currentColor" fill="none" x="82.68" y="108" width="171.72" height="24" rx="5"></rect><text fill="currentColor" x="168.54000000000002" y="124" text-anchor="middle">open</text></g></g> <text fill="currentColor" x="82.68" y="18" text-anchor="middle">held 60s: oncall notified</text> <text fill="currentColor" x="254.4" y="18" text-anchor="middle">clears: closes</text></svg>

The condition has to hold for its whole duration before the policy opens an incident and notifies the channel. When the errors stop, the incident closes.

## Count log lines

A logs-based metric turns matching log entries into a metric you can chart and alert on:

```typescript
const errors = yield* GCP.Logging.Metric("Errors", {
  filter: 'resource.type = "cloud_run_revision" AND severity >= ERROR',
  description: "error log entries",
});
```

## Route logs elsewhere

A sink copies matching entries to another destination, such as a log bucket or BigQuery:

```typescript
yield* GCP.Logging.Sink("ErrorArchive", {
  destination:
    "logging.googleapis.com/projects/my-project/locations/global/buckets/_Default",
  filter: "severity >= ERROR",
});
```

## Where next

- [Observability](../../testing/observability.md) — tracing Effect programs.
- [Monitoring reference](https://alchemy.run/providers/gcp/monitoring#alertpolicy) and [Logging reference](https://alchemy.run/providers/gcp/logging#metric) — every prop.
