---
url: https://alchemy.run/fly/federated-identity
title: "Federated identity"
description: "How a Fly Service reaches AWS and GCP with the OIDC tokens its Machines sign, instead of a stored access key or service account key."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A Fly Service often needs a resource on another cloud: an S3 bucket, a Pub/Sub topic. Its Machines can prove which App they belong to, so AWS and GCP can trust them directly. There is no access key or service account key to create, store or rotate.

You bind the resource the same way you would on a Lambda or Cloud Run. Federation is what Alchemy picks for a Fly Service.

## Machines sign their own identity

Every Machine serves part of the Machines API on a Unix socket, `/.fly/api`, already authenticated as that Machine. A process on it can ask for an OIDC token without holding any secret:

```sh
curl --unix-socket /.fly/api -X POST http://localhost/v1/tokens/oidc \
  -d '{"aud": "sts.amazonaws.com"}'
```

The token is signed by your organization’s issuer, `https://oidc.fly.io/<org>`, and names the Machine in its subject:

```json
{
  "iss": "https://oidc.fly.io/my-org",
  "aud": "sts.amazonaws.com",
  "sub": "my-org:api-app:3d8d9e1a2b3c4d",
  "app_name": "api-app",
  "exp": 1762300800
}
```

A token lasts 10 minutes. The socket answers only root, the default user in the images Alchemy builds. A Service whose image switches to another user fails its first cross-cloud call with `OidcTokenError`.

## Call AWS

Bind an S3 bucket to a Service:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as Fly from "@alchemy.run/fly";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export const Uploads = AWS.S3.Bucket("Uploads");

export default class Api extends Fly.Service<Api>()(
  "Api",
  { main: import.meta.url, port: 3000 },
  Effect.gen(function* () {
    const putObject = yield* AWS.S3.PutObject(yield* Uploads);
    return {
      fetch: putObject({ Key: "hello.txt", Body: "hello" }).pipe(
        Effect.as(HttpServerResponse.text("stored")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(AWS.S3.PutObjectHttp)),
) {}
```

The binding registers your organization’s issuer as an IAM OIDC provider, and creates a role that holds `s3:PutObject` on the bucket. The role trusts tokens whose subject starts with the Service’s App, so every Machine of the App can assume it and no other App can:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.fly.io/my-org"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": { "oidc.fly.io/my-org:aud": "sts.amazonaws.com" },
    "StringLike": { "oidc.fly.io/my-org:sub": "my-org:api-app:*" }
  }
}
```

At runtime the Machine mints a token and trades it for the role’s credentials:

<svg viewBox="0 0 668 289" style="max-width: 768px" role="img" aria-label="The Machine asks Fly who it is, and AWS checks Fly's signature and the App in the subject. Shortly before the credentials expire, it mints a fresh token and assumes the role again."><defs><marker id="alc-seq-arrow-ets19u" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-ets19u" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="285"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="82" y="51" text-anchor="middle">Fly Machine</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="285"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">/.fly/api</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">Fly</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="285"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="42.5" text-anchor="middle">AWS STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="586" y1="59" x2="586" y2="285"></line><rect stroke="currentColor" fill="none" x="520" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="586" y="36" text-anchor="middle">Uploads</text> <text fill="currentColor" x="586" y="51" text-anchor="middle">S3 bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-ets19u)"></line><rect stroke="currentColor" fill="none" x="56.400000000000006" y="66" width="219.2" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">OIDC token for sts.amazonaws.com</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="125" x2="88" y2="125" marker-end="url(#alc-seq-arrow-ets19u)"></line><rect stroke="currentColor" fill="none" x="92.7" y="106" width="146.6" height="15" rx="3"></rect><text fill="currentColor" x="166" y="118" text-anchor="middle">signed token · 10 min</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="165" x2="412" y2="165" marker-end="url(#alc-seq-arrow-ets19u)"></line><rect stroke="currentColor" fill="none" x="163.5" y="146" width="173" height="15" rx="3"></rect><text fill="currentColor" x="250" y="158" text-anchor="middle">AssumeRoleWithWebIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="205" x2="88" y2="205" marker-end="url(#alc-seq-arrow-ets19u)"></line><rect stroke="currentColor" fill="none" x="163.5" y="186" width="173" height="15" rx="3"></rect><text fill="currentColor" x="250" y="198" text-anchor="middle">role credentials · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="245" x2="580" y2="245" marker-end="url(#alc-seq-arrow-ets19u)"></line><rect stroke="currentColor" fill="none" x="300.3" y="226" width="67.4" height="15" rx="3"></rect><text fill="currentColor" x="334" y="238" text-anchor="middle">PutObject</text></g></svg> <svg viewBox="0 0 322 288" style="max-width: 370px" role="img" aria-label="The Machine asks Fly who it is, and AWS checks Fly's signature and the App in the subject. Shortly before the credentials expire, it mints a fresh token and assumes the role again."><defs><marker id="alc-seq-arrow-n-ets19u" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-ets19u" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="58" x2="38.75" y2="284"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="38.75" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="38.75" y="37" text-anchor="middle">Fly</text> <text fill="currentColor" x="38.75" y="50" text-anchor="middle">Machine</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="58" x2="120.25" y2="284"></line><rect stroke="currentColor" fill="none" x="83.5" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="120.25" y="22" text-anchor="middle">/.fly/api</text> <text fill="currentColor" x="120.25" y="37" text-anchor="middle">Fly</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="58" x2="201.75" y2="284"></line><rect stroke="currentColor" fill="none" x="165" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="201.75" y="35" text-anchor="middle">AWS STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="58" x2="283.25" y2="284"></line><rect stroke="currentColor" fill="none" x="246.5" y="2" width="73.5" height="56" rx="8"></rect><text fill="currentColor" x="283.25" y="22" text-anchor="middle">Uploads</text> <text fill="currentColor" x="283.25" y="37" text-anchor="middle">S3 bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="84" x2="114.25" y2="84" marker-end="url(#alc-seq-arrow-n-ets19u)"></line><rect stroke="currentColor" fill="none" x="-2" y="65" width="219.2" height="15" rx="3"></rect><text fill="currentColor" x="107.6" y="77" text-anchor="middle">OIDC token for sts.amazonaws.com</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="124" x2="44.75" y2="124" marker-end="url(#alc-seq-arrow-n-ets19u)"></line><rect stroke="currentColor" fill="none" x="6.200000000000003" y="105" width="146.6" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="117" text-anchor="middle">signed token · 10 min</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="164" x2="195.75" y2="164" marker-end="url(#alc-seq-arrow-n-ets19u)"></line><rect stroke="currentColor" fill="none" x="33.75" y="145" width="173" height="15" rx="3"></rect><text fill="currentColor" x="120.25" y="157" text-anchor="middle">AssumeRoleWithWebIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="204" x2="44.75" y2="204" marker-end="url(#alc-seq-arrow-n-ets19u)"></line><rect stroke="currentColor" fill="none" x="33.75" y="185" width="173" height="15" rx="3"></rect><text fill="currentColor" x="120.25" y="197" text-anchor="middle">role credentials · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="244" x2="277.25" y2="244" marker-end="url(#alc-seq-arrow-n-ets19u)"></line><rect stroke="currentColor" fill="none" x="127.30000000000001" y="225" width="67.4" height="15" rx="3"></rect><text fill="currentColor" x="161" y="237" text-anchor="middle">PutObject</text></g></svg>

The Machine asks Fly who it is, and AWS checks Fly's signature and the App in the subject. Shortly before the credentials expire, it mints a fresh token and assumes the role again.

## One OIDC provider per organization

IAM allows one OIDC provider per issuer URL in an account. Fly’s issuer is per organization, so every stack that binds a Fly Service in the same account and organization shares one provider. The provider only says that Fly’s signatures are trusted; each role’s condition decides which App may use it.

Destroying a stack leaves the provider in place, because other stacks may still use it. Delete it yourself once no stack needs it:

```sh
aws iam delete-open-id-connect-provider \
  --open-id-connect-provider-arn arn:aws:iam::123456789012:oidc-provider/oidc.fly.io/my-org
```

## Call GCP

Bind a Pub/Sub topic the same way:

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

The binding creates a service account that holds the publisher role on the topic. The stack gets a workload identity pool whose provider trusts your organization’s issuer and maps each token to its App. The service account lets that one App impersonate it:

```plaintext
principalSet://iam.googleapis.com/projects/123456789/locations/global/workloadIdentityPools/<pool>/attribute.fly_app/api-app
```

At runtime the Machine mints a token addressed to the pool’s provider, trades it at Google STS for a federated token, and trades that for the service account’s access token, which lasts an hour.

A pool isn’t tied to an issuer, so each stack has its own and destroys it with the stack.

## Calls between Services

A Service calling another Fly Service needs no token either. Fly’s proxy signs every private request with a `Fly-Src` header naming the calling App, and a Service answers only the Apps that bind it. See [Connect Services](compute/connecting-services.md#security-model).

## Sprites hold a key

A [Sprite](compute/sprites.md) is not a Machine and has no `/.fly/api` socket, so it can’t mint tokens. A Sprite that binds an AWS or GCP resource gets a key for an identity that may do only what that Sprite needs, deployed as a secret. The binding code is the same.

## New grants take a few minutes

AWS usually accepts a new role’s trust within seconds. A GCP grant to a federated principal can take a few minutes to land, and until it does calls fail with a permission error. Retry the first call after a deploy.

## Where next

- [Identity Federation](../infrastructure-as-effects/identity-federation.md) — how federation works across every cloud Alchemy supports.
- [AWS federated identity](../aws/security/federated-identity.md) and [GCP federated identity](../gcp/security/federated-identity.md) — the same trust from the target cloud’s side.
- [Connect Services](compute/connecting-services.md) — Services that call each other over Fly’s private network.
