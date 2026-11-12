---
url: https://alchemy.run/aws/observability/cloudwatch
title: "CloudWatch"
description: "How CloudWatch collects logs and metrics from every AWS service, and how alarms and dashboards declared next to your resources watch them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

CloudWatch is where AWS reports on itself. Every service publishes **metrics** to it without being asked: a Function’s invocations and errors, a Queue’s depth, a Table’s throttled requests. Every Function writes its **logs** to it. **Alarms** watch metrics and tell you when something crosses a line, and **dashboards** put the metrics you care about on one screen.

## Metrics come from every service

A **metric** is a series of numbers over time, identified by a **namespace**, a name and **dimensions**. Lambda reports `Errors` in the `AWS/Lambda` namespace, with a `FunctionName` dimension for each Function. SQS reports `ApproximateNumberOfMessagesVisible` per `QueueName`.

You don’t publish these. They exist as soon as the resource does, so an alarm or a dashboard can watch them from the first deploy.

## Logs go to log groups

Everything a Function writes to its output lands in a **log group** named after it, `/aws/lambda/<name>`. ECS tasks get a log group too. Set how long they’re kept on the runtime, or they’re kept forever:

```typescript
{ main: import.meta.url, logging: { retention: "2 weeks" } }
```

Read a runtime’s logs from your terminal:

```sh
alchemy logs Api --tail
```

## Alarm on a metric

An **alarm** watches one metric. When the metric crosses a threshold for long enough, the alarm moves to the `ALARM` state:

```typescript
const api = yield* Api;

const errors = yield* AWS.CloudWatch.Alarm("ApiErrors", {
  Namespace: "AWS/Lambda",
  MetricName: "Errors",
  Dimensions: [{ Name: "FunctionName", Value: api.functionName }],
  Statistic: "Sum",
  Period: 60,
  EvaluationPeriods: 1,
  Threshold: 1,
  ComparisonOperator: "GreaterThanOrEqualToThreshold",
  TreatMissingData: "notBreaching",
});
```

The dimension comes from the Function itself, so the alarm follows it across renames and stages. `TreatMissingData: "notBreaching"` keeps a Function that nobody’s calling out of the `INSUFFICIENT_DATA` state.

## Be told when it fires

An alarm acts by publishing to an [SNS Topic](../messaging/sns.md). Subscribe your email, a chat webhook or a pager to the Topic:

<svg viewBox="-4 -13.614495755427528 788 150.22899151085505" style="max-width: 906px" role="img" aria-label="An alarm watches one metric and changes state when it crosses the threshold. Moving into ALARM runs its actions, here a publish to the Alerts Topic."><defs><marker id="alc-sm-arrow-aowsye" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 204 63 Q 264 63 320 63" marker-end="url(#alc-sm-arrow-aowsye)"></path><path stroke="currentColor" fill="none" d="M 444.3333333333333 86 Q 548 148.2 648.2366949638165 88.0579830217101" marker-end="url(#alc-sm-arrow-aowsye)"></path><path stroke="currentColor" fill="none" d="M 651.6666666666666 40 Q 548 -22.200000000000003 447.7633050361835 37.94201697828989" marker-end="url(#alc-sm-arrow-aowsye)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-aowsye)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="67" text-anchor="middle">INSUFFICIENT_DATA</text></g> <g><rect stroke="currentColor" fill="none" x="324" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="406" y="67" text-anchor="middle">OK</text></g> <g><rect stroke="currentColor" fill="none" x="608" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="690" y="61" text-anchor="middle">ALARM</text> <text fill="currentColor" x="690" y="76" text-anchor="middle">publishes to Alerts</text></g> <g><rect stroke="currentColor" fill="none" x="218.4" y="53" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="263" y="66" text-anchor="middle">data arrives</text></g> <g><rect stroke="currentColor" fill="none" x="469.5425070742874" y="107.61449575542753" width="155.2" height="17" rx="4"></rect><text fill="currentColor" x="547.1425070742874" y="120.61449575542753" text-anchor="middle">Errors ≥ 1 in a minute</text></g> <g><rect stroke="currentColor" fill="none" x="510.85749292571245" y="-1.6144957554275283" width="76" height="17" rx="4"></rect><text fill="currentColor" x="548.8574929257124" y="11.385504244572472" text-anchor="middle">back under</text></g></svg> <svg viewBox="-10.852531828189711 -4 344.9050636563794 378" style="max-width: 397px" role="img" aria-label="An alarm watches one metric and changes state when it crosses the threshold. Moving into ALARM runs its actions, here a publish to the Alerts Topic."><defs><marker id="alc-sm-arrow-n-aowsye" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 122 86 Q 122 132 122 174" marker-end="url(#alc-sm-arrow-n-aowsye)"></path><path stroke="currentColor" fill="none" d="M 144.08 224 Q 188.24 270 146.85012731275887 313.1144507158762" marker-end="url(#alc-sm-arrow-n-aowsye)"></path><path stroke="currentColor" fill="none" d="M 99.92 316 Q 55.760000000000005 270 97.14987268724114 226.8855492841238" marker-end="url(#alc-sm-arrow-n-aowsye)"></path><g><g><circle stroke="currentColor" fill="none" cx="122" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="122" y1="20" x2="122" y2="36" marker-end="url(#alc-sm-arrow-n-aowsye)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="67" text-anchor="middle">INSUFFICIENT_DATA</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="205" text-anchor="middle">OK</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="337" text-anchor="middle">ALARM</text> <text fill="currentColor" x="122" y="352" text-anchor="middle">publishes to Alerts</text></g> <g><rect stroke="currentColor" fill="none" x="77.4" y="121" width="89.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="122" y="134" text-anchor="middle">data arrives</text></g> <g><rect stroke="currentColor" fill="none" x="166.85253182818974" y="259.27861267896907" width="155.2" height="17" rx="4"></rect><text fill="currentColor" x="171.85253182818974" y="272.27861267896907" text-anchor="start">Errors ≥ 1 in a minute</text></g> <g><rect stroke="currentColor" fill="none" x="1.1474681718102886" y="260.72138732103093" width="76" height="17" rx="4"></rect><text fill="currentColor" x="72.14746817181029" y="273.72138732103093" text-anchor="end">back under</text></g></svg>

An alarm watches one metric and changes state when it crosses the threshold. Moving into ALARM runs its actions, here a publish to the Alerts Topic.

```typescript
const alerts = yield* AWS.SNS.Topic("Alerts");

const errors = yield* AWS.CloudWatch.Alarm("ApiErrors", {
  Namespace: "AWS/Lambda",
  MetricName: "Errors",
  Dimensions: [{ Name: "FunctionName", Value: api.functionName }],
  Statistic: "Sum",
  Period: 60,
  EvaluationPeriods: 1,
  Threshold: 1,
  ComparisonOperator: "GreaterThanOrEqualToThreshold",
  AlarmActions: [alerts.topicArn],
});
```

Because alarms are declared in the Stack, each stage gets its own. Point production’s at a pager and every other stage’s at nothing.

## Put metrics on a dashboard

A **dashboard** is a grid of widgets. Its body is a plain object, built from the Function’s name once it’s known:

```typescript
yield* AWS.CloudWatch.Dashboard("Api", {
  DashboardBody: api.functionName.pipe(
    Output.map((functionName) => ({
      widgets: [
        {
          type: "metric",
          width: 12,
          height: 6,
          properties: {
            title: "Invocations and errors",
            stat: "Sum",
            period: 300,
            metrics: [
              ["AWS/Lambda", "Invocations", "FunctionName", functionName],
              ["AWS/Lambda", "Errors", "FunctionName", functionName],
            ],
          },
        },
      ],
    })),
  ),
});
```

Widgets can also show text, a grid of alarm states, or a Logs Insights query. A change to a dashboard is a change in a pull request, reviewed like any other.

## Combine alarms

One failure often trips several alarms at once. A **composite alarm** combines their states with a rule, so you’re paged once:

```typescript
yield* AWS.CloudWatch.CompositeAlarm("ApiUnhealthy", {
  AlarmRule: Output.interpolate\`ALARM("${errors.alarmName}") OR ALARM("${latency.alarmName}")\`,
  AlarmActions: [alerts.topicArn],
});
```

Alarm names are generated per stage, so build the rule from each alarm’s `alarmName`.

## Your own telemetry

CloudWatch’s built-in metrics describe your resources from the outside. To see inside your code, its spans, structured logs and custom metrics, export OpenTelemetry from your Effect program. See [Observability](../../testing/observability.md), and [Axiom](../../axiom.md) for a backend declared in the same Stack.
