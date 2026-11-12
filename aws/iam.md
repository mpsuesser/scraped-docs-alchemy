---
url: https://alchemy.run/aws/iam
title: "IAM and bindings"
description: "How IAM identities, roles, trust, resource-based policies and tag-based access (ABAC) work on AWS, and how Alchemy bindings write each runtime's least-privilege policies from the calls its code makes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

IAM is the part of AWS that answers one question on every request: may this identity do this action on this resource? Nothing is allowed until a policy says so.

## Every request has an identity

When your code calls `s3:PutObject`, AWS first works out who is asking. The caller signs each request with credentials, and the credentials belong to an **identity**.

There are two kinds. An **IAM user** has long-lived keys of its own. An **IAM role** has none: a trusted principal **assumes** it and receives temporary credentials that expire within hours.

Prefer roles everywhere. A person signs in through IAM Identity Center and assumes a role in the account they work in. A Lambda Function, an ECS task or an EC2 instance runs as a role AWS assumes for it. CI assumes a role through OIDC. No long-lived key is ever written down.

## A policy lists what an identity may do

A **policy** is a JSON document of statements. Each statement names an effect, the actions it covers and the resources it applies to:

```json
{
  "Effect": "Allow",
  "Action": ["sqs:SendMessage"],
  "Resource": "arn:aws:sqs:us-east-1:123456789012:myapp-jobs-a1b2c3"
}
```

Actions are `service:Operation` pairs, one per API call. Resources are ARNs, and a `*` in one matches many: `arn:aws:s3:::myapp-files-a1b2c3/*` is every object in one Bucket.

AWS denies by default. A request is allowed only if some statement allows it and no statement denies it. So the safe policy is the smallest one: exactly the actions the code calls, on exactly the resources it calls them on. That’s **least privilege**.

## A role is trusted, then assumed

A role has two policies. Its **permissions** say what it may do. Its **trust policy** says who may assume it.

<svg viewBox="0 0 724 290" style="max-width: 833px" role="img" aria-label="A role holds two policies. The trust policy lets Lambda assume the role. The permissions let the role send to one Queue."><defs><marker id="alc-dag-arrow-vii854" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="272" y="24" width="180" height="242" rx="10" ry="10"></rect><text fill="currentColor" x="284" y="41" text-anchor="start">Execution role</text></g> <g><path d="M 172 82 L 282 82" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-vii854)"></path><text fill="currentColor" x="227" y="76" text-anchor="middle">assumes</text></g> <g><path d="M 436 218 L 546 218" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-vii854)"></path></g><g transform="translate(24, 50)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Lambda</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">AWS service</text></g> <g transform="translate(288, 50)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">trust policy</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">who may assume it</text></g> <g transform="translate(288, 186)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">permissions</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">SendMessage on Jobs</text></g> <g transform="translate(552, 186)"><rect stroke="currentColor" fill="none" width="148" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="74" y="28" text-anchor="middle" dominant-baseline="middle">Jobs</text> <text fill="currentColor" x="74" y="46" text-anchor="middle" dominant-baseline="middle">SQS Queue</text></g></svg> <svg viewBox="0 0 401 356" style="max-width: 461px" role="img" aria-label="A role holds two policies. The trust policy lets Lambda assume the role. The permissions let the role send to one Queue."><defs><marker id="alc-dag-arrow-7vz8fn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="10" y="120" width="381" height="106" rx="10" ry="10"></rect><text fill="currentColor" x="106" y="137" text-anchor="start">Execution role</text></g> <g><path d="M 96 74 L 96 140" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-7vz8fn)"></path><text fill="currentColor" x="104" y="103" text-anchor="start">assumes</text></g> <g><path d="M 305 210 L 305 276" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-7vz8fn)"></path></g><g transform="translate(26, 10)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Lambda</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">AWS service</text></g> <g transform="translate(26, 146)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">trust policy</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">who may assume it</text></g> <g transform="translate(235, 146)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">permissions</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">SendMessage on Jobs</text></g> <g transform="translate(235, 282)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Jobs</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">SQS Queue</text></g></svg>

A role holds two policies. The trust policy lets Lambda assume the role. The permissions let the role send to one Queue.

A Lambda Function’s execution role trusts `lambda.amazonaws.com`, so the Lambda service can assume it on the Function’s behalf. Every time your code runs, it carries the role’s temporary credentials, and every call it makes is checked against the role’s permissions.

## A resource can have a policy too

So far every policy has been attached to an identity, saying what it may do. Many resources carry a policy of their own, saying who may reach them. Buckets, Queues, Topics, KMS keys, Secrets and Lambda Functions all do. This is a **resource-based policy**, and its statements name a **principal**: the account, role or service the statement is about.

```json
{
  "Effect": "Allow",
  "Principal": { "Service": "cloudfront.amazonaws.com" },
  "Action": ["s3:GetObject"],
  "Resource": "arn:aws:s3:::myapp-site-a1b2c3/*",
  "Condition": { "StringEquals": { "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E1A2B3" } }
}
```

This one lets CloudFront read the site’s files, but only on behalf of one distribution. The condition matters: without it, any distribution in any account could read the Bucket.

Within one account, either kind of policy can allow a request. Across accounts, both must: the caller’s identity policy has to allow the call, and the resource’s policy has to allow the caller’s account.

<svg viewBox="0 0 700 401" style="max-width: 805px" role="img" aria-label="A call across accounts passes two checks, one in each account, and both must allow it. Within one account, either is enough. An explicit deny in either always wins."><defs><marker id="alc-seq-arrow-4y0nlz" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-4y0nlz" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="59" x2="86" y2="397"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="86" y="36" text-anchor="middle">Role</text> <text fill="currentColor" x="86" y="51" text-anchor="middle">account 1111</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="262" y1="59" x2="262" y2="397"></line><rect stroke="currentColor" fill="none" x="192" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="262" y="36" text-anchor="middle">identity policy</text> <text fill="currentColor" x="262" y="51" text-anchor="middle">account 1111</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="438" y1="59" x2="438" y2="397"></line><rect stroke="currentColor" fill="none" x="368" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="438" y="36" text-anchor="middle">bucket policy</text> <text fill="currentColor" x="438" y="51" text-anchor="middle">account 2222</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="614" y1="59" x2="614" y2="397"></line><rect stroke="currentColor" fill="none" x="544" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="614" y="36" text-anchor="middle">Bucket</text> <text fill="currentColor" x="614" y="51" text-anchor="middle">account 2222</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="85" x2="256" y2="85" marker-end="url(#alc-seq-arrow-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="107.30000000000001" y="66" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="174" y="78" text-anchor="middle">may I s3:GetObject?</text></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="262" y1="125" x2="92" y2="125" marker-end="url(#alc-seq-arrow-4y0nlz)"></line> <rect stroke="currentColor" fill="none" x="153.5" y="106" width="41" height="15" rx="3"></rect><text fill="currentColor" x="174" y="118" text-anchor="middle">allow</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="165" x2="432" y2="165" marker-end="url(#alc-seq-arrow-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="228.3" y="146" width="67.4" height="15" rx="3"></rect><text fill="currentColor" x="262" y="158" text-anchor="middle">GetObject</text></g> <g><rect stroke="currentColor" x="336.9" y="193" width="202.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="438" y="209" text-anchor="middle">does it trust account 1111?</text></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="438" y1="241" x2="608" y2="241" marker-end="url(#alc-seq-arrow-4y0nlz)"></line> <rect stroke="currentColor" fill="none" x="505.5" y="222" width="41" height="15" rx="3"></rect><text fill="currentColor" x="526" y="234" text-anchor="middle">allow</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="614" y1="281" x2="92" y2="281" marker-end="url(#alc-seq-arrow-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="313" y="262" width="74" height="15" rx="3"></rect><text fill="currentColor" x="350" y="274" text-anchor="middle">the object</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="16" y1="329" x2="684" y2="329"></line><rect stroke="currentColor" fill="none" x="16" y="320" width="132.60000000000002" height="18" rx="4"></rect><text fill="currentColor" x="22" y="333">within one account</text></g> <g><rect stroke="currentColor" x="204" y="349" width="292" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="350" y="365" text-anchor="middle">either policy allowing is enough</text></g></svg> <svg viewBox="0 0 322 415" style="max-width: 370px" role="img" aria-label="A call across accounts passes two checks, one in each account, and both must allow it. Within one account, either is enough. An explicit deny in either always wins."><defs><marker id="alc-seq-arrow-n-4y0nlz" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-4y0nlz" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="73" x2="38.75" y2="411"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="38.75" y="22" text-anchor="middle">Role</text> <text fill="currentColor" x="38.75" y="52" text-anchor="middle">account</text> <text fill="currentColor" x="38.75" y="65" text-anchor="middle">1111</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="73" x2="120.25" y2="411"></line><rect stroke="currentColor" fill="none" x="83.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="120.25" y="22" text-anchor="middle">identity</text> <text fill="currentColor" x="120.25" y="37" text-anchor="middle">policy</text> <text fill="currentColor" x="120.25" y="52" text-anchor="middle">account</text> <text fill="currentColor" x="120.25" y="65" text-anchor="middle">1111</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="73" x2="201.75" y2="411"></line><rect stroke="currentColor" fill="none" x="165" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="201.75" y="22" text-anchor="middle">bucket</text> <text fill="currentColor" x="201.75" y="37" text-anchor="middle">policy</text> <text fill="currentColor" x="201.75" y="52" text-anchor="middle">account</text> <text fill="currentColor" x="201.75" y="65" text-anchor="middle">2222</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="73" x2="283.25" y2="411"></line><rect stroke="currentColor" fill="none" x="246.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="283.25" y="22" text-anchor="middle">Bucket</text> <text fill="currentColor" x="283.25" y="52" text-anchor="middle">account</text> <text fill="currentColor" x="283.25" y="65" text-anchor="middle">2222</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="99" x2="114.25" y2="99" marker-end="url(#alc-seq-arrow-n-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="12.800000000000004" y="80" width="133.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="92" text-anchor="middle">may I s3:GetObject?</text></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="139" x2="44.75" y2="139" marker-end="url(#alc-seq-arrow-n-4y0nlz)"></line> <rect stroke="currentColor" fill="none" x="59" y="120" width="41" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="132" text-anchor="middle">allow</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="179" x2="195.75" y2="179" marker-end="url(#alc-seq-arrow-n-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="86.55" y="160" width="67.4" height="15" rx="3"></rect><text fill="currentColor" x="120.25" y="172" text-anchor="middle">GetObject</text></g> <g><rect stroke="currentColor" x="100.65" y="207" width="202.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="201.75" y="223" text-anchor="middle">does it trust account 1111?</text></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="255" x2="277.25" y2="255" marker-end="url(#alc-seq-arrow-n-4y0nlz)"></line> <rect stroke="currentColor" fill="none" x="222" y="236" width="41" height="15" rx="3"></rect><text fill="currentColor" x="242.5" y="248" text-anchor="middle">allow</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="295" x2="44.75" y2="295" marker-end="url(#alc-seq-arrow-n-4y0nlz)"></line><rect stroke="currentColor" fill="none" x="124" y="276" width="74" height="15" rx="3"></rect><text fill="currentColor" x="161" y="288" text-anchor="middle">the object</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="2" y1="343" x2="320" y2="343"></line><rect stroke="currentColor" fill="none" x="2" y="334" width="132.60000000000002" height="18" rx="4"></rect><text fill="currentColor" x="8" y="347">within one account</text></g> <g><rect stroke="currentColor" x="43.400000000000006" y="363" width="235.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="161" y="379" text-anchor="middle">either policy allowing is enough</text></g></svg>

A call across accounts passes two checks, one in each account, and both must allow it. Within one account, either is enough. An explicit deny in either always wins.

Services that call your resources, like CloudFront, EventBridge, SNS or S3 notifications, have no identity policy in your account. A resource policy is the only way to let them in.

## Bindings write the policy

Writing that policy by hand means keeping two lists in step: the calls in the code, and the statements in the role. Add a call and forget the statement, and the call fails in production. Remove a call and forget the statement, and the role keeps a permission nobody uses.

A **binding** keeps them in step. It’s one line in the runtime’s Construction phase that returns a typed client and adds the matching statement to the runtime’s role:

```typescript
const sendMessage = yield* SQS.SendMessage(Jobs);
```

Deploy adds `sqs:SendMessage` on the `Jobs` Queue’s ARN to the Function’s role. Call the client while the Function runs:

```typescript
yield* sendMessage({ MessageBody: JSON.stringify({ id }) });
```

The client comes from the binding’s Layer, `SQS.SendMessageHttp`, which the Function provides:

```typescript
export default class Api extends AWS.Lambda.Function<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const sendMessage = yield* SQS.SendMessage(Jobs);
    return {
      fetch: sendMessage({ MessageBody: "hello" }).pipe(
        Effect.as(HttpServerResponse.text("queued")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(SQS.SendMessageHttp)),
) {}
```

Each binding is one IAM statement. Bind `S3.GetObject` as well, and the role gains `s3:GetObject` on that Bucket’s objects. Delete a binding, and the next deploy removes its statement.

## Bindings are scoped to one resource

A binding takes the resource it grants access to, so its statement names that resource’s ARN and nothing else:

```typescript
const getObject = yield* S3.GetObject(Files);
const putObject = yield* S3.PutObject(Uploads);
```

The Function may read from `Files` and write to `Uploads`. It can’t write to `Files`, and it can’t touch any other Bucket in the account, even one created by the same Stack.

## The role comes with the runtime

You never declare a Lambda Function’s execution role. Deploying the Function creates it, with the Lambda trust policy, permission to write its own logs, and one statement per binding. Destroying the Function deletes it.

The role’s name and ARN are on the Function, for anything outside the Function that needs them:

```typescript
const api = yield* Api;
return { role: api.roleArn };
```

ECS tasks, EKS workloads and EC2 instances work the same way. A binding on any of them adds its statement to the role that runs it. See [Choosing a runtime](compute/choosing-a-runtime.md).

## Bindings write resource policies too

When the caller is a service rather than your code, the binding writes the resource’s policy instead of a role’s. You don’t declare these statements, and each one carries the condition that limits it to the one caller:

A Website’s CloudFront distribution gets `s3:GetObject` in its Bucket’s policy, for that distribution only. An EventBridge rule that targets a Queue gets `sqs:SendMessage` in the Queue’s policy, for that rule only. A rule, schedule or Bucket notification that targets a Function gets a **Lambda permission**, the resource policy that lets the service invoke it. An event source that polls, like a Queue or a Stream, instead gets read permissions on the Function’s role.

Remove the website, rule or subscription, and the next deploy removes its statement.

## Open a resource to another account

A resource policy is also how you share a resource with another account. Add your own statements to a Bucket’s `policy`, and they’re merged with the ones bindings contribute:

```typescript
export const Reports = AWS.S3.Bucket("Reports", {
  bucketName: "acme-reports",
  policy: [
    {
      Effect: "Allow",
      Principal: { AWS: "arn:aws:iam::222222222222:root" },
      Action: ["s3:GetObject"],
      Resource: ["arn:aws:s3:::acme-reports/*"],
    },
  ],
});
```

Name the Bucket yourself here, so the statement can name its ARN. The other account still has to allow its own roles to make the call. Queues and KMS keys take a `policy` the same way. To let another account or a service invoke a Function, declare a permission:

```typescript
yield* AWS.Lambda.Permission("PartnerInvoke", {
  functionName: api.functionName,
  action: "lambda:InvokeFunction",
  principal: "222222222222",
});
```

## Grant by tag with ABAC

Bindings grant access to named resources: this Function may write to this Bucket. That’s the right model when the code knows what it calls. Sometimes access follows a rule instead: anyone on the payments team may read the payments team’s tables, including ones created next month.

**Attribute-based access control** (ABAC) writes that rule with tags. A statement’s condition compares a tag on the resource with a tag on the caller, so one statement covers every matching resource, current and future:

```typescript
const analysts = yield* AWS.IAM.Role("PaymentsAnalysts", {
  assumeRolePolicyDocument: {
    Version: "2012-10-17",
    Statement: [
      {
        Effect: "Allow",
        Principal: { AWS: "arn:aws:iam::123456789012:root" },
        Action: ["sts:AssumeRole", "sts:TagSession"],
      },
    ],
  },
  tags: { team: "payments" },
  inlinePolicies: {
    ReadTeamTables: {
      Version: "2012-10-17",
      Statement: [
        {
          Effect: "Allow",
          Action: ["dynamodb:GetItem", "dynamodb:Query"],
          Resource: "*",
          Condition: {
            StringEquals: { "aws:ResourceTag/team": "${aws:PrincipalTag/team}" },
          },
        },
      ],
    },
  },
});
```

Tag the resources the rule should cover:

```typescript
export const Ledger = AWS.DynamoDB.Table("Ledger", {
  partitionKey: "id",
  attributes: { id: "S" },
  tags: { team: "payments" },
});
```

Alchemy already tags everything it creates with `alchemy::stack`, `alchemy::stage` and `alchemy::id`, so you can scope a rule to one stage without adding tags of your own:

```json
{ "StringEquals": { "aws:ResourceTag/alchemy::stage": "prod" } }
```

Not every action supports tag conditions. Check the service’s entry in the [AWS service authorization reference](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html) before relying on one. And keep tags under control: whoever can change a tag can change who has access.

## Declare a role for anything else

Some identities aren’t runtimes: the role CI assumes to deploy, a role another service assumes, a role you hand to a partner account. Declare those with `AWS.IAM.Role`, giving its trust policy and its permissions:

```typescript
const role = yield* AWS.IAM.Role("DeployRole", {
  assumeRolePolicyDocument: {
    Version: "2012-10-17",
    Statement: [
      {
        Effect: "Allow",
        Principal: { Federated: oidc.openIDConnectProviderArn },
        Action: ["sts:AssumeRoleWithWebIdentity"],
        Condition: {
          StringLike: { "token.actions.githubusercontent.com:sub": "repo:your-org/your-repo:*" },
        },
      },
    ],
  },
  managedPolicyArns: ["arn:aws:iam::aws:policy/AdministratorAccess"],
});
```

This one lets GitHub Actions in one repository deploy without a stored key. The [CI tutorial](tutorial/part-5.md) builds it step by step.

## Trust another cloud

A runtime on another cloud can be trusted the same way. A Cloud Run service binds an SQS Queue exactly as a Lambda does, and the binding creates a role that trusts that one Google service account. There’s no access key to create, store or rotate. See [Federated identity](security/federated-identity.md).

## Keep secrets out of policies

IAM decides who may call AWS. It doesn’t hold your third-party API keys. Pass those to a runtime from your `.env` through `effect/Config`, or keep them in Secrets Manager when other resources share or rotate them. See [Secrets & env](security/secrets-env.md).
