---
url: https://alchemy.run/aws/networking/custom-domains
title: "Custom domains"
description: "How a domain reaches a site or API on AWS through a hosted zone, a TLS certificate and DNS records, and how the domain prop sets them up on Websites, Lambda Functions and load balancers."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A custom domain needs three things. **DNS** has to send the name to the thing serving it. A **certificate** has to prove to browsers that you own the name. And the thing serving it, CloudFront, a load balancer or API Gateway, has to accept the name and present the certificate.

On AWS, DNS is **Route 53** and certificates come from **ACM**.

## Zone, certificate, record

A **hosted zone** holds the DNS records for a domain. A **certificate** for a name is issued once you prove you control it, by adding a validation record ACM gives you to the zone. A record for the name then points at the thing serving it.

<svg viewBox="0 0 470 290" style="max-width: 541px" role="img" aria-label="The hosted zone holds two records: one proves you own the name, so ACM issues the certificate, and one points the name at the distribution, which presents the certificate."><defs><marker id="alc-dag-arrow-g1rdot" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="24" y="24" width="180.4" height="242" rx="10" ry="10"></rect><text fill="currentColor" x="192.4" y="41" text-anchor="end">Hosted zone example.com</text></g> <g><path d="M 185 82 L 295 82" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-g1rdot)"></path><text fill="currentColor" x="240" y="76" text-anchor="middle">points at</text></g> <g><path d="M 185 218 L 295 218" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-g1rdot)"></path><text fill="currentColor" x="240" y="212" text-anchor="middle">validates</text></g> <g><path d="M 373.5 186 L 373.5 120" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-g1rdot)"></path><text fill="currentColor" x="381.5" y="149" text-anchor="start">presents</text></g> <g transform="translate(40, 50)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">app.example.com</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">alias record</text></g> <g transform="translate(40, 186)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">_x.example.com</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">validation CNAME</text></g> <g transform="translate(301, 50)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">Distribution</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">CloudFront</text></g> <g transform="translate(301, 186)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">certificate</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">ACM</text></g></svg>

The hosted zone holds two records: one proves you own the name, so ACM issues the certificate, and one points the name at the distribution, which presents the certificate.

An **alias record** is Route 53’s way to point a name at an AWS resource. It works at the zone’s apex, `example.com` itself, where an ordinary CNAME can’t.

Certificates for CloudFront must be in `us-east-1`, wherever the rest of the app runs. Certificates for a load balancer or a regional API must be in that resource’s region.

## Give a resource a domain

Websites, Lambda Functions and ECS load balancers take a `domain` prop. Name the domain, and Alchemy requests the certificate, validates it, attaches it and creates the records, `A` and `AAAA` aliases for websites and load balancers:

```typescript
const site = yield* AWS.Website.StaticSite("Site", {
  path: "./dist",
  domain: { name: "app.example.com", aliases: ["www.app.example.com"] },
});
```

The zone is found by walking up the name’s labels until one matches a hosted zone in the account. Pass `hostedZoneId` to name it yourself. The site’s `url` becomes `https://app.example.com`.

A Lambda Function’s URL can’t serve a custom name. Give the Function a `domain`, and Alchemy puts an HTTP API in front of it with a regional certificate. Its `fetch` handler works unchanged:

```typescript
const api = yield* AWS.Lambda.Function("Api", { main: "./src/handler.ts", domain: "api.example.com" });
```

An ECS Service with `loadBalancer` takes a `domain` too, on its load balancer.

## Use DNS hosted somewhere else

When the domain’s DNS is on Cloudflare or Hetzner, give the `domain` that host’s DNS adapter:

```typescript
const site = yield* AWS.Website.StaticSite("Site", {
  path: "./dist",
  domain: { name: "app.example.com", dns: Cloudflare.DNS.Adapter() },
});
```

The certificate is still issued by ACM, and its validation records and the name’s CNAME are written to the other host instead. Add `Cloudflare.providers()` next to `AWS.providers()` in the Stack. See [DNS adapters](../../infrastructure-as-code/dns-adapters.md).

To manage DNS by hand, set `dns: false` and pass a certificate you already have as `cert`. Without access to the DNS, a new certificate can’t be validated.

## Create a hosted zone

To bring a domain into Route 53, declare its zone:

```typescript
const zone = yield* AWS.Route53.HostedZone("Zone", { name: "example.com" });
```

Then set the zone’s `nameServers` as the domain’s name servers at your registrar. From then on, Route 53 answers for the domain.

This always creates a new zone, since Route 53 allows several zones with the same name. To manage a zone that already exists, pass its id:

```typescript
const zone = yield* AWS.Route53.HostedZone("Zone", {
  name: "example.com",
  hostedZoneId: "Z0123456789ABCDEFGHIJ",
});
```

To only reference it, without managing it, use `AWS.Route53.HostedZone.lookup({ id })`.

## Build it from the pieces

The `domain` prop is made of resources you can declare yourself. A certificate, validated in the zone:

```typescript
const cert = yield* AWS.ACM.Certificate("Cert", {
  domainName: "app.example.com",
  subjectAlternativeNames: ["www.app.example.com"],
  hostedZoneId: zone.id,
});
```

Deploy writes the validation records and waits until the certificate is issued, so `cert.certificateArn` is ready to use. For a regional resource outside `us-east-1`, set `region`.

Alias records pointing a name at a distribution, one per address family:

```typescript
for (const type of ["A", "AAAA"] as const) {
  yield* AWS.Route53.Record(\`Alias-${type}\`, {
    hostedZoneId: zone.id,
    name: "app.example.com",
    type,
    aliasTarget: { hostedZoneId: distribution.hostedZoneId, dnsName: distribution.domainName },
  });
}
```

CloudFront serves IPv6, so without the `AAAA` record IPv6 clients get no answer for the name. The alias target’s `hostedZoneId` is the distribution’s own, not your zone’s. A REST API on API Gateway takes a `DomainName` with the certificate and a `BasePathMapping` onto its stage. See [API Gateway](../apis/api-gateway.md).

## Changing a domain

Certificates can’t be changed in place. Adding a name or changing the domain replaces the certificate: a new one is requested and validated, and the resource is moved onto it.

Validation records stay in the zone after a certificate is deleted, because ACM uses the same record for every certificate on that name.
