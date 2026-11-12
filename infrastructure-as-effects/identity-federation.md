---
url: https://alchemy.run/infrastructure-as-effects/identity-federation
title: "Identity Federation"
description: "How a binding reaches another cloud with the identity the host's platform already signs, instead of a long-lived key, and which runtimes and clouds support it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A [Binding](binding.md) needs credentials to call its resource. On the resource’s own cloud that’s easy: a Lambda calls DynamoDB as its IAM role, and the binding puts the permission on that role. Nothing is created or stored.

A binding can also cross clouds. Here a Cloud Run service sends to an SQS queue:

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

The service has no AWS identity. The obvious fix is an access key stored as a secret. A key never expires on its own, works from anywhere, and has to be rotated by hand. Alchemy uses **identity federation** instead.

## Trust the identity the platform signs

Cloud Run already proves who a service is. Its metadata server hands the service an ID token, signed by Google, that names the service’s account. AWS can be told to trust that token.

So the binding creates an IAM role whose trust policy accepts Google’s tokens for that one service account, and grants `sqs:SendMessage` on the role. At runtime the service trades a fresh token for the role’s credentials:

<svg viewBox="0 0 700 289" style="max-width: 805px" role="img" aria-label="The service proves who it is with a token its platform signed, and leaves with short-lived AWS credentials. No key exists anywhere."><defs><marker id="alc-seq-arrow-hvpqfq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-hvpqfq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="59" x2="86" y2="285"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="86" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="86" y="51" text-anchor="middle">Cloud Run service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="262" y1="59" x2="262" y2="285"></line><rect stroke="currentColor" fill="none" x="192" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="262" y="36" text-anchor="middle">Metadata server</text> <text fill="currentColor" x="262" y="51" text-anchor="middle">Google</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="438" y1="59" x2="438" y2="285"></line><rect stroke="currentColor" fill="none" x="368" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="438" y="42.5" text-anchor="middle">AWS STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="614" y1="59" x2="614" y2="285"></line><rect stroke="currentColor" fill="none" x="544" y="16" width="140" height="43" rx="8"></rect><text fill="currentColor" x="614" y="36" text-anchor="middle">Jobs</text> <text fill="currentColor" x="614" y="51" text-anchor="middle">SQS queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="85" x2="256" y2="85" marker-end="url(#alc-seq-arrow-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="143.6" y="66" width="60.8" height="15" rx="3"></rect><text fill="currentColor" x="174" y="78" text-anchor="middle">ID token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="262" y1="125" x2="92" y2="125" marker-end="url(#alc-seq-arrow-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="130.4" y="106" width="87.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="174" y="118" text-anchor="middle">signed token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="165" x2="432" y2="165" marker-end="url(#alc-seq-arrow-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="175.5" y="146" width="173" height="15" rx="3"></rect><text fill="currentColor" x="262" y="158" text-anchor="middle">AssumeRoleWithWebIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="438" y1="205" x2="92" y2="205" marker-end="url(#alc-seq-arrow-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="175.5" y="186" width="173" height="15" rx="3"></rect><text fill="currentColor" x="262" y="198" text-anchor="middle">role credentials · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="86" y1="245" x2="608" y2="245" marker-end="url(#alc-seq-arrow-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="309.7" y="226" width="80.6" height="15" rx="3"></rect><text fill="currentColor" x="350" y="238" text-anchor="middle">SendMessage</text></g></svg> <svg viewBox="0 0 322 303" style="max-width: 370px" role="img" aria-label="The service proves who it is with a token its platform signed, and leaves with short-lived AWS credentials. No key exists anywhere."><defs><marker id="alc-seq-arrow-n-hvpqfq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-hvpqfq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="73" x2="38.75" y2="299"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="38.75" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="38.75" y="52" text-anchor="middle">Cloud Run</text> <text fill="currentColor" x="38.75" y="65" text-anchor="middle">service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="73" x2="120.25" y2="299"></line><rect stroke="currentColor" fill="none" x="83.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="120.25" y="22" text-anchor="middle">Metadata</text> <text fill="currentColor" x="120.25" y="37" text-anchor="middle">server</text> <text fill="currentColor" x="120.25" y="52" text-anchor="middle">Google</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="73" x2="201.75" y2="299"></line><rect stroke="currentColor" fill="none" x="165" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="201.75" y="42.5" text-anchor="middle">AWS STS</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="283.25" y1="73" x2="283.25" y2="299"></line><rect stroke="currentColor" fill="none" x="246.5" y="2" width="73.5" height="71" rx="8"></rect><text fill="currentColor" x="283.25" y="22" text-anchor="middle">Jobs</text> <text fill="currentColor" x="283.25" y="52" text-anchor="middle">SQS queue</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="99" x2="114.25" y2="99" marker-end="url(#alc-seq-arrow-n-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="49.1" y="80" width="60.8" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="92" text-anchor="middle">ID token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="120.25" y1="139" x2="44.75" y2="139" marker-end="url(#alc-seq-arrow-n-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="35.900000000000006" y="120" width="87.19999999999999" height="15" rx="3"></rect><text fill="currentColor" x="79.5" y="132" text-anchor="middle">signed token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="179" x2="195.75" y2="179" marker-end="url(#alc-seq-arrow-n-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="33.75" y="160" width="173" height="15" rx="3"></rect><text fill="currentColor" x="120.25" y="172" text-anchor="middle">AssumeRoleWithWebIdentity</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="201.75" y1="219" x2="44.75" y2="219" marker-end="url(#alc-seq-arrow-n-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="33.75" y="200" width="173" height="15" rx="3"></rect><text fill="currentColor" x="120.25" y="212" text-anchor="middle">role credentials · 1 hour</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="38.75" y1="259" x2="277.25" y2="259" marker-end="url(#alc-seq-arrow-n-hvpqfq)"></line><rect stroke="currentColor" fill="none" x="120.7" y="240" width="80.6" height="15" rx="3"></rect><text fill="currentColor" x="161" y="252" text-anchor="middle">SendMessage</text></g></svg>

The service proves who it is with a token its platform signed, and leaves with short-lived AWS credentials. No key exists anywhere.

The credentials last an hour. Shortly before they expire the binding asks for a new token and assumes the role again. Nothing long-lived ever leaves Google.

The binding code is the one you would write on a Lambda. Alchemy sees where the host runs and picks federation for you.

## Scope the trust to one workload

A trust that names only the issuer would let every service in Google assume the role. So the trust always names the one workload: Cloud Run’s service account by its unique ID, a Fly App by the subject in its token, a Lambda by the role it runs as. Another service, even one in the same project or organization, can’t use it.

Where the target cloud needs an account-wide anchor first, such as an IAM OIDC provider for Fly’s issuer, Alchemy creates it once and keeps it when a stack is destroyed, because other stacks may depend on it.

## Which runtimes federate

A runtime can federate when its platform signs an identity the other cloud can check.

Into AWS, Cloud Run services, jobs and worker pools and Cloud Functions send Google ID tokens. Fly Services send Fly Machine OIDC tokens. See [AWS federated identity](../aws/security/federated-identity.md).

Into GCP, Lambda functions, ECS tasks, EC2 instances, App Runner services, Batch jobs and EKS pods sign an STS `GetCallerIdentity` request with their role, which Google checks against AWS. Fly Services send Fly Machine OIDC tokens. See [GCP federated identity](../gcp/security/federated-identity.md).

Fly Services do both from the same token. See [Fly federated identity](../fly/federated-identity.md).

## When the platform signs nothing

Cloudflare Workers, Railway services and Fly Sprites have no workload identity token today, and a GKE pod can’t get a token AWS accepts. On those hosts the binding creates a key for an identity that may do only what that one host needs, and deploys it as a secret. A Worker’s key, for example, can assume only the Worker’s own role.

You write the same binding either way.

## Calls between runtimes

The same rule applies when one runtime calls another. The callee checks an identity the platform signed, not a shared secret.

A Lambda invoke is authorized by IAM. A private Cloud Run service checks the caller’s Google ID token against its invoker role. A Fly Service checks the `Fly-Src` header Fly’s proxy signs, which names the calling App, and answers only the Apps that bind it. See [Connect Services](../fly/compute/connecting-services.md#security-model).

## Deploying from CI

A deploy needs credentials too. CI platforms sign tokens for each run, so a workflow can assume a deploy role without storing a key. See [AWS with GitHub OIDC](../environments/ci.md#aws-with-github-oidc-recommended).

## New grants take a few minutes

A fresh trust isn’t usable right away. AWS usually accepts a new role within seconds. A GCP grant to a federated principal can take a few minutes to land. Until it does, calls fail with a permission error, so retry the first call after a deploy.

## Where next

- [AWS federated identity](../aws/security/federated-identity.md) — Cloud Run and Fly Services calling AWS.
- [GCP federated identity](../gcp/security/federated-identity.md) — Lambda, ECS, EC2, Batch, EKS and Fly Services calling GCP.
- [Fly federated identity](../fly/federated-identity.md) — Fly Machine OIDC tokens, and how Fly Services call AWS and GCP.
- [Layers](layers.md) — hide bindings behind a service interface.
