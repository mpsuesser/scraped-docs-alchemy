---
url: https://alchemy.run/aws/email/receiving
title: "Email receiving"
description: "Receive inbound email with SES — rule sets and rules that store mail in S3, fan out to SNS, invoke a Lambda, or bounce it, plus IP filters and the active rule set pointer."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

SES can receive mail as well as send it. Point your domain’s MX record at SES, and every message sent to an address on the domain arrives at AWS, where your **receipt rules** decide what happens to it: store it in S3, notify an SNS Topic, invoke a Function, or bounce it.

## Rule sets, rules and actions

Rules live in a **rule set**. An account can hold many rule sets, but SES evaluates only the **active** one. Within it, rules run in order. Each rule matches messages by recipient and runs its **actions** in order: store the raw message in a Bucket, publish to a Topic, invoke a Function, bounce, add a header, or stop.

<svg viewBox="0 0 668 317" style="max-width: 768px" role="img" aria-label="Filters run first. Then the active rule set runs its rules in order, and each matching rule runs its actions in order."><defs><marker id="alc-seq-arrow-p5zutg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-p5zutg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="313"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">mail server</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">the sender's</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="313"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">SES</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">active rule set</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="313"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Inbox</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">S3 Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586" y1="59" x2="586" y2="313"></line><rect stroke="currentColor" fill="none" x="520" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="586" y="36" text-anchor="middle">Triage</text> <text fill="currentColor" x="586" y="51" text-anchor="middle">Lambda Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-p5zutg)"></line><rect stroke="currentColor" fill="none" x="69.60000000000001" y="66" width="192.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">mail for support@example.com</text></g> <g><rect stroke="currentColor" x="115.9" y="113" width="268.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="129" text-anchor="middle">receipt filters: allow or block by IP</text></g> <g><rect stroke="currentColor" x="162.10000000000002" y="149" width="175.79999999999998" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="165" text-anchor="middle">rule 1 matches support@</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="197" x2="412" y2="197" marker-end="url(#alc-seq-arrow-p5zutg)"></line><rect stroke="currentColor" fill="none" x="280.5" y="178" width="107" height="15" rx="3"></rect><text fill="currentColor" x="334" y="190" text-anchor="middle">action 1: store</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="237" x2="580" y2="237" marker-end="url(#alc-seq-arrow-p5zutg)"></line><rect stroke="currentColor" fill="none" x="361.2" y="218" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="418" y="230" text-anchor="middle">action 2: invoke</text></g> <g><rect stroke="currentColor" x="115.9" y="265" width="268.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="281" text-anchor="middle">then rule 2, unless an action stopped</text></g></svg> <svg viewBox="-2 0 324 331" style="max-width: 373px" role="img" aria-label="Filters run first. Then the active rule set runs its rules in order, and each matching rule runs its actions in order."><defs><marker id="alc-seq-arrow-n-p5zutg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-p5zutg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="73" x2="38.75" y2="327"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="38.75" y="22" text-anchor="middle">mail</text> <text fill="currentColor" x="38.75" y="37" text-anchor="middle">server</text> <text fill="currentColor" x="38.75" y="52" text-anchor="middle">the</text> <text fill="currentColor" x="38.75" y="65" text-anchor="middle">sender's</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="73" x2="120.25" y2="327"></line><rect stroke="currentColor" fill="none" x="83.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="120.25" y="22" text-anchor="middle">SES</text> <text fill="currentColor" x="120.25" y="52" text-anchor="middle">active</text> <text fill="currentColor" x="120.25" y="65" text-anchor="middle">rule set</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="73" x2="201.75" y2="327"></line><rect stroke="currentColor" fill="none" x="165" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="201.75" y="22" text-anchor="middle">Inbox</text> <text fill="currentColor" x="201.75" y="52" text-anchor="middle">S3 Bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="73" x2="283.25" y2="327"></line><rect stroke="currentColor" fill="none" x="246.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="283.25" y="22" text-anchor="middle">Triage</text> <text fill="currentColor" x="283.25" y="52" text-anchor="middle">Lambda</text> <text fill="currentColor" x="283.25" y="65" text-anchor="middle">Function</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="99" x2="114.25" y2="99" marker-end="url(#alc-seq-arrow-n-p5zutg)"></line><rect stroke="currentColor" fill="none" x="-2" y="80" width="192.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="94.39999999999999" y="92" text-anchor="middle">mail for support@example.com</text></g> <g><rect stroke="currentColor" x="2" y="127" width="268.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="136.1" y="143" text-anchor="middle">receipt filters: allow or block by IP</text></g> <g><rect stroke="currentColor" x="32.35000000000001" y="163" width="175.79999999999998" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="120.25" y="179" text-anchor="middle">rule 1 matches support@</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="211" x2="195.75" y2="211" marker-end="url(#alc-seq-arrow-n-p5zutg)"></line><rect stroke="currentColor" fill="none" x="107.5" y="192" width="107" height="15" rx="3"></rect><text fill="currentColor" x="161" y="204" text-anchor="middle">action 1: store</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="251" x2="277.25" y2="251" marker-end="url(#alc-seq-arrow-n-p5zutg)"></line><rect stroke="currentColor" fill="none" x="144.95" y="232" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="201.75" y="244" text-anchor="middle">action 2: invoke</text></g> <g><rect stroke="currentColor" x="2" y="279" width="268.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="136.1" y="295" text-anchor="middle">then rule 2, unless an action stopped</text></g></svg>

Filters run first. Then the active rule set runs its rules in order, and each matching rule runs its actions in order.

**Receipt filters** run before any rule, allowing or blocking mail by the sending server’s IP address.

## Create the Stack

The snippets below run inside a Stack’s `Effect.gen` body:

```typescript
import * as Alchemy from "alchemy";
import * as AWS from "@alchemy.run/aws";
import * as SES from "@alchemy.run/aws/SES";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "InboundMail",
  { providers: AWS.providers(), state: AWS.state() },
  Effect.gen(function* () {
    // resources go here
    return {};
  }),
);
```

## Create a rule set

A rule set starts empty — it’s just the ordered container the rules hang off:

```typescript
const ruleSet = yield* SES.ReceiptRuleSet("Inbound", {});
```

Omitting `ruleSetName` lets Alchemy generate a deterministic name from the app, stage, and logical id. Nothing is evaluated against inbound mail until a rule set is made active (below).

## Store inbound mail in S3

The running example is a support inbox: store every raw message in a bucket, then notify a topic that new mail arrived. Start with the S3 action:

```typescript
const bucket = yield* AWS.S3.Bucket("InboundMail", {
  forceDestroy: true,
});

const inbound = yield* SES.ReceiptRule("Support", {
  ruleSetName: ruleSet.ruleSetName,
  recipients: ["support@example.com"],
  actions: [
    {
      S3Action: {
        BucketName: bucket.bucketName,
        ObjectKeyPrefix: "support/",
      },
    },
  ],
});
```

SES writes the full raw MIME message to the bucket under the prefix, keyed by message id. The bucket’s policy must allow the SES service principal (`ses.amazonaws.com`) to `s3:PutObject` — SES validates this when the rule is created, so add it to the bucket before the rule.

## Fan out to SNS

Actions run in array order. Add an `SNSAction` after the S3 action so SES publishes a notification once the message is stored:

```typescript
const topic = yield* AWS.SNS.Topic("InboundNotifications", {});

const inbound = yield* SES.ReceiptRule("Support", {
  ruleSetName: ruleSet.ruleSetName,
  recipients: ["support@example.com"],
  actions: [
    {
      S3Action: {
        BucketName: bucket.bucketName,
        ObjectKeyPrefix: "support/",
      },
    },
    { SNSAction: { TopicArn: topic.topicArn, Encoding: "UTF-8" } },
  ],
});
```

Anything subscribed to the topic — a queue, a Lambda, an HTTP endpoint — now fans out from the single notification. The topic’s access policy must allow `ses.amazonaws.com` to `sns:Publish`. (An `SNSAction` publishes a notification, not the message body; for large messages the S3 copy is the source of truth.)

## Invoke a Lambda on inbound mail

A `LambdaAction` hands the matched message to a function. Point it at the function’s ARN and choose `Event` (async, fire-and-forget) or `RequestResponse` (synchronous, lets the function influence delivery):

```typescript
const rule = yield* SES.ReceiptRule("Process", {
  ruleSetName: ruleSet.ruleSetName,
  recipients: ["support@example.com"],
  actions: [
    {
      LambdaAction: {
        FunctionArn: processor.functionArn,
        InvocationType: "Event",
      },
    },
  ],
});
```

See [Lambda](../compute/lambda.md) for standing up the `processor` function itself. SES’s permission to invoke it is granted the same way as S3 and SNS — the function’s resource policy must allow `ses.amazonaws.com` to `lambda:InvokeFunction`.

## Order rules with after

Rules evaluate top to bottom. The `after` prop pins a rule immediately behind another one in the set. Here a catch-all bounce runs after the support rule, rejecting mail to unknown recipients:

```typescript
const bounce = yield* SES.ReceiptRule("BounceUnknown", {
  ruleSetName: ruleSet.ruleSetName,
  after: inbound.ruleName,
  actions: [
    {
      BounceAction: {
        SmtpReplyCode: "550",
        Message: "Mailbox does not exist",
        Sender: "mailer-daemon@example.com",
      },
    },
  ],
});
```

A rule with no `recipients` matches everything, so ordering it last makes it a fallthrough. Omit `after` to place a rule at the top of the set.

## Activate the rule set

Rules do nothing until their set is the account’s active one. `ActiveReceiptRuleSet` is the pointer SES evaluates against every inbound message:

```typescript
yield* SES.ActiveReceiptRuleSet("Active", {
  ruleSetName: ruleSet.ruleSetName,
});
```

Only one rule set is active per account and region. Re-pointing this resource at a different set switches receiving over in place. Destroying it deactivates receiving — but only if the account is still pointed at the set this resource activated; if something else has since taken over, the destroy is a no-op.

## Filter senders by IP

`ReceiptFilter` is an account-level allow/block list on the source IP of inbound connections, applied before any rule runs. Block filters win over allow filters:

```typescript
yield* SES.ReceiptFilter("BlockBadActors", {
  ipFilter: { policy: "Block", cidr: "203.0.113.0/24" },
});
```

`cidr` takes a single address (`203.0.113.7`) or a range (`203.0.113.0/24`). Filters have no update API, so changing the policy or range replaces the filter.

## Bounce a message from a Lambda

When a `LambdaAction` invokes your function synchronously, the function can bounce the message itself. `SES.SendBounce` is a runtime binding — an account-level operation with no resource to scope to, so it’s bound with no argument and its implementation is provided as `SES.SendBounceHttp`:

```typescript
// Construction — inside the Lambda's Effect.gen body
const sendBounce = yield* SES.SendBounce();
```

At runtime, generate the bounce from the id of the message SES delivered to you:

```typescript
yield* sendBounce({
  OriginalMessageId: messageId,
  BounceSender: "mailer-daemon@example.com",
  BouncedRecipientInfoList: [
    { Recipient: "nobody@example.com", BounceType: "DoesNotExist" },
  ],
});
```

Provide the implementation on the function with `Effect.provide(SES.SendBounceHttp)`. You can only bounce a message within 24 hours of receiving it, and only mail SES actually received for you.

## Where next

- [Lambda](../compute/lambda.md) — the function a `LambdaAction` invokes.
- [S3](../data/s3.md) — the bucket inbound mail lands in.
- [SNS](../messaging/sns.md) — fan the notification out to queues and functions.

Reference:

- [ReceiptRuleSet API reference](https://alchemy.run/providers/aws/ses#receiptruleset)
- [ReceiptRule API reference](https://alchemy.run/providers/aws/ses#receiptrule)
- [ActiveReceiptRuleSet API reference](https://alchemy.run/providers/aws/ses#activereceiptruleset)
- [ReceiptFilter API reference](https://alchemy.run/providers/aws/ses#receiptfilter)
- [SendBounce API reference](https://alchemy.run/providers/aws/ses#sendbounce)
