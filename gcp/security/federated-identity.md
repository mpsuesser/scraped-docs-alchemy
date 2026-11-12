---
url: https://alchemy.run/gcp/security/federated-identity
title: "Federated identity"
description: "How a Lambda or Fly Service reaches GCP, and a Cloud Run service reaches AWS, through workload identity federation instead of long-lived keys."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A runtime on one cloud often needs a resource on the other: a Lambda publishing to a Pub/Sub topic, or a Cloud Run service sending to an SQS queue. When the runtime already runs as an identity of its own cloud, Alchemy lets the other cloud trust that identity. There is no service account key or access key to create, store, or rotate.

You write the binding the same way as on a native runtime. Federation is what Alchemy picks for you.

[Identity Federation](../../infrastructure-as-effects/identity-federation.md) covers how federation works across every cloud.

## Call GCP from a Lambda

A Lambda runs as an IAM role. Bind a Pub/Sub topic to it like you would on Cloud Run:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export const Events = GCP.PubSub.Topic("Events", {});

export default class Worker extends AWS.Lambda.Function<Worker>()(
  "Worker",
  { main: import.meta.url, functionUrl: true },
  Effect.gen(function* () {
    const publish = yield* GCP.PubSub.Publish(yield* Events);
    return {
      fetch: publish({ body: { messages: [{ data: btoa("hello") }] } }).pipe(
        Effect.as(HttpServerResponse.text("published")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(GCP.PubSub.PublishHttp)),
) {}
```

The binding creates a service account for the Lambda and grants it the publisher role on the topic, as it would for any runtime. Instead of a key, the Lambda’s role may impersonate that account.

The trust goes through a workload identity pool with an AWS provider for your account. The stack shares one pool, so every AWS runtime in the stack uses it. The provider maps each caller to the role it assumed, and the account grants `roles/iam.workloadIdentityUser` to that one role:

```plaintext
principalSet://iam.googleapis.com/projects/123456789/locations/global/workloadIdentityPools/<pool>/attribute.aws_role/arn:aws:sts::111122223333:assumed-role/<role>
```

<svg viewBox="0 0 1002 369" style="max-width: 1152px" role="img" aria-label="The Lambda proves who it is with its own AWS credentials, and leaves with a short-lived Google token for the service account the binding created. No key is stored anywhere."><defs><marker id="alc-seq-arrow-nbdxch" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-nbdxch" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="98.6" y1="59" x2="98.6" y2="365"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="165.2" height="43" rx="8"></rect><text fill="currentColor" x="98.6" y="36" text-anchor="middle">Lambda</text> <text fill="currentColor" x="98.6" y="51" text-anchor="middle">IAM role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="299.79999999999995" y1="59" x2="299.79999999999995" y2="365"></line><rect stroke="currentColor" fill="none" x="217.19999999999996" y="16" width="165.2" height="43" rx="8"></rect><text fill="currentColor" x="299.79999999999995" y="36" text-anchor="middle">Google STS</text> <text fill="currentColor" x="299.79999999999995" y="51" text-anchor="middle">workload identity pool</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="501" y1="59" x2="501" y2="365"></line><rect stroke="currentColor" fill="none" x="418.4" y="16" width="165.2" height="43" rx="8"></rect><text fill="currentColor" x="501" y="42.5" text-anchor="middle">AWS STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="702.1999999999999" y1="59" x2="702.1999999999999" y2="365"></line><rect stroke="currentColor" fill="none" x="619.5999999999999" y="16" width="165.2" height="43" rx="8"></rect><text fill="currentColor" x="702.1999999999999" y="36" text-anchor="middle">IAM Credentials</text> <text fill="currentColor" x="702.1999999999999" y="51" text-anchor="middle">Google</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="903.4" y1="59" x2="903.4" y2="365"></line><rect stroke="currentColor" fill="none" x="820.8" y="16" width="165.2" height="43" rx="8"></rect><text fill="currentColor" x="903.4" y="36" text-anchor="middle">Events</text> <text fill="currentColor" x="903.4" y="51" text-anchor="middle">Pub/Sub topic</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="98.6" y1="85" x2="293.79999999999995" y2="85" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="116" y="66" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="199.2" y="78" text-anchor="middle">signed GetCallerIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="299.79999999999995" y1="125" x2="495" y2="125" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="337" y="106" width="126.8" height="15" rx="3"></rect><text fill="currentColor" x="400.4" y="118" text-anchor="middle">replay the request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="501" y1="165" x2="305.79999999999995" y2="165" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="313.9" y="146" width="173" height="15" rx="3"></rect><text fill="currentColor" x="400.4" y="158" text-anchor="middle">caller: the Lambda's role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="299.79999999999995" y1="205" x2="104.6" y2="205" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="145.7" y="186" width="107" height="15" rx="3"></rect><text fill="currentColor" x="199.2" y="198" text-anchor="middle">federated token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="98.6" y1="245" x2="696.1999999999999" y2="245" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="294.09999999999997" y="226" width="212.6" height="15" rx="3"></rect><text fill="currentColor" x="400.4" y="238" text-anchor="middle">impersonate the service account</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="702.1999999999999" y1="285" x2="104.6" y2="285" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="327.09999999999997" y="266" width="146.6" height="15" rx="3"></rect><text fill="currentColor" x="400.4" y="278" text-anchor="middle">access token · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="98.6" y1="325" x2="897.4" y2="325" marker-end="url(#alc-seq-arrow-nbdxch)"></line><rect stroke="currentColor" fill="none" x="473.9" y="306" width="54.199999999999996" height="15" rx="3"></rect><text fill="currentColor" x="501" y="318" text-anchor="middle">publish</text></g></svg> <svg viewBox="0 0 322 396" style="max-width: 370px" role="img" aria-label="The Lambda proves who it is with its own AWS credentials, and leaves with a short-lived Google token for the service account the binding created. No key is stored anywhere."><defs><marker id="alc-seq-arrow-n-nbdxch" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-nbdxch" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="30.6" y1="86" x2="30.6" y2="392"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="57.2" height="84" rx="8"></rect><text fill="currentColor" x="30.6" y="22" text-anchor="middle">Lambda</text> <text fill="currentColor" x="30.6" y="52" text-anchor="middle">IAM</text> <text fill="currentColor" x="30.6" y="65" text-anchor="middle">role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="95.80000000000001" y1="86" x2="95.80000000000001" y2="392"></line><rect stroke="currentColor" fill="none" x="67.20000000000002" y="2" width="57.2" height="84" rx="8"></rect><text fill="currentColor" x="95.80000000000001" y="22" text-anchor="middle">Google</text> <text fill="currentColor" x="95.80000000000001" y="37" text-anchor="middle">STS</text> <text fill="currentColor" x="95.80000000000001" y="52" text-anchor="middle">workload</text> <text fill="currentColor" x="95.80000000000001" y="65" text-anchor="middle">identity</text> <text fill="currentColor" x="95.80000000000001" y="78" text-anchor="middle">pool</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="86" x2="161" y2="392"></line><rect stroke="currentColor" fill="none" x="132.4" y="2" width="57.2" height="84" rx="8"></rect><text fill="currentColor" x="161" y="41.5" text-anchor="middle">AWS</text> <text fill="currentColor" x="161" y="56.5" text-anchor="middle">STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="226.20000000000002" y1="86" x2="226.20000000000002" y2="392"></line><rect stroke="currentColor" fill="none" x="197.60000000000002" y="2" width="57.2" height="84" rx="8"></rect><text fill="currentColor" x="226.20000000000002" y="22" text-anchor="middle">IAM</text> <text fill="currentColor" x="226.20000000000002" y="37" text-anchor="middle">Credentials</text> <text fill="currentColor" x="226.20000000000002" y="52" text-anchor="middle">Google</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="291.40000000000003" y1="86" x2="291.40000000000003" y2="392"></line><rect stroke="currentColor" fill="none" x="262.8" y="2" width="57.2" height="84" rx="8"></rect><text fill="currentColor" x="291.40000000000003" y="22" text-anchor="middle">Events</text> <text fill="currentColor" x="291.40000000000003" y="52" text-anchor="middle">Pub/Sub</text> <text fill="currentColor" x="291.40000000000003" y="65" text-anchor="middle">topic</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="30.6" y1="112" x2="89.80000000000001" y2="112" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="-2" y="93" width="166.39999999999998" height="15" rx="3"></rect><text fill="currentColor" x="81.19999999999999" y="105" text-anchor="middle">signed GetCallerIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="95.80000000000001" y1="152" x2="155" y2="152" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="65" y="133" width="126.8" height="15" rx="3"></rect><text fill="currentColor" x="128.4" y="145" text-anchor="middle">replay the request</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="192" x2="101.80000000000001" y2="192" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="41.900000000000006" y="173" width="173" height="15" rx="3"></rect><text fill="currentColor" x="128.4" y="185" text-anchor="middle">caller: the Lambda's role</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="95.80000000000001" y1="232" x2="36.6" y2="232" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="9.700000000000003" y="213" width="107" height="15" rx="3"></rect><text fill="currentColor" x="63.2" y="225" text-anchor="middle">federated token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="30.6" y1="272" x2="220.20000000000002" y2="272" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="22.10000000000001" y="253" width="212.6" height="15" rx="3"></rect><text fill="currentColor" x="128.4" y="265" text-anchor="middle">impersonate the service account</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="226.20000000000002" y1="312" x2="36.6" y2="312" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="55.10000000000001" y="293" width="146.6" height="15" rx="3"></rect><text fill="currentColor" x="128.4" y="305" text-anchor="middle">access token · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="30.6" y1="352" x2="285.40000000000003" y2="352" marker-end="url(#alc-seq-arrow-n-nbdxch)"></line><rect stroke="currentColor" fill="none" x="133.90000000000003" y="333" width="54.199999999999996" height="15" rx="3"></rect><text fill="currentColor" x="161.00000000000003" y="345" text-anchor="middle">publish</text></g></svg>

The Lambda proves who it is with its own AWS credentials, and leaves with a short-lived Google token for the service account the binding created. No key is stored anywhere.

At runtime the Lambda signs an STS `GetCallerIdentity` request with its own role credentials and sends it to Google STS. Google replays the request against AWS to learn which role is calling, and returns a federated token. The Lambda trades that for the service account’s token, which lasts an hour and is exchanged again before it expires.

## Call GCP from a Fly Service

A Fly Service runs on Fly Machines, and every Machine can mint an OIDC token signed by `https://oidc.fly.io/<org>`. Bind the topic the same way:

```typescript
import * as Fly from "@alchemy.run/fly";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export const Events = GCP.PubSub.Topic("Events", {});

export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const publish = yield* GCP.PubSub.Publish(yield* Events);
    return {
      fetch: publish({ body: { messages: [{ data: btoa("hello") }] } }).pipe(
        Effect.as(HttpServerResponse.text("published")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(GCP.PubSub.PublishHttp)),
) {}
```

The stack gets a second pool, whose provider trusts Fly’s issuer for your organization. The provider maps each token to its App, and the account grants `roles/iam.workloadIdentityUser` to that one App:

```plaintext
principalSet://iam.googleapis.com/projects/123456789/locations/global/workloadIdentityPools/<pool>/attribute.fly_app/<app>
```

At runtime the Machine asks its `/.fly/api` socket for a token addressed to the provider and sends it to Google STS, then trades the federated token for the service account’s token. The socket answers only root, the default user in the images Alchemy builds. See [Fly federated identity](../../fly/federated-identity.md) for the tokens themselves.

## Call AWS from Cloud Run

A Cloud Run service runs as a Google service account. Bind an SQS queue to it:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export const Jobs = AWS.SQS.Queue("Jobs");

export default class Api extends GCP.Run.Service<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const send = yield* AWS.SQS.SendMessage(Jobs);
    return {
      fetch: send({ MessageBody: "hello" }).pipe(
        Effect.as(HttpServerResponse.text("sent")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(AWS.SQS.SendMessageHttp)),
) {}
```

The binding creates an IAM role that trusts Google’s ID tokens for the service’s runtime account, matched by the account’s unique ID. At runtime the service asks its metadata server for an ID token and trades it for the role’s credentials with `AssumeRoleWithWebIdentity`. The credentials last an hour and are assumed again, with a fresh token, shortly before they expire.

## Which runtimes federate

Lambda functions, ECS tasks and services, EC2 instances and launch templates, App Runner services, Batch jobs, EKS pods, and Fly Services reach GCP this way. Cloud Run services, jobs, and worker pools, Cloud Functions, and Fly Services reach AWS this way.

Runtimes without an identity the other cloud can trust still get a key: a Cloudflare Worker, a Railway service, a Fly Sprite, and a GKE pod calling AWS. The key belongs to an identity that may do only what that one host needs. The binding code is the same either way.

## New grants take a few minutes

A federated grant is not usable right away. A grant to a workload identity principal can take a few minutes to land, and until it does the Lambda’s calls fail with `iam.serviceAccounts.getAccessToken` denied. Retry the first call after a deploy.

See [IAM & bindings](iam.md) for the roles each binding grants.
