---
url: https://alchemy.run/acme/what-is-acme
title: "What is ACME?"
description: "The concepts behind ACME certificates — certificate authorities, accounts, orders, challenges and DNS-01 validation, certificates and keys, renewal and revocation — and how @alchemy.run/acme models each of them."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A TLS certificate proves to a browser that a server really is `example.com`. **ACME** is the protocol certificate authorities like Let’s Encrypt and ZeroSSL use to issue those certificates automatically: you prove you control a domain, and the authority signs a certificate for it, for free, in under a minute.

Take one certificate: a wildcard for `*.example.com`, validated through a Cloudflare zone and installed on a Fly app. An account asks the authority for a certificate. The authority sets a challenge, which is answered with a DNS record. Once it sees the record, the authority signs the certificate, which goes to whatever serves the domain.

<svg viewBox="0 0 479.4 268" style="max-width: 551px" role="img" aria-label="The authority signs only after it finds the TXT record in your zone, which only the domain's owner could publish."><defs><marker id="alc-bd-arrow-fq4myj" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="198.6" height="228" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="264.79999999999995" y="156" width="198.60000000000002" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="196.6" y1="78" x2="278.79999999999995" y2="78" marker-end="url(#alc-bd-arrow-fq4myj)"></line><text fill="currentColor" x="239.7" y="70" text-anchor="middle">order</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="196.6" y1="210" x2="278.79999999999995" y2="210" marker-end="url(#alc-bd-arrow-fq4myj)"></line><text fill="currentColor" x="239.7" y="202" text-anchor="middle">publish</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="364.09999999999997" y1="102" x2="364.09999999999997" y2="182" marker-end="url(#alc-bd-arrow-fq4myj)"></line><text fill="currentColor" x="374.09999999999997" y="148">checks</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">your deploy</text> <text fill="currentColor" x="276.79999999999995" y="175" text-anchor="start">example.com</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="75" text-anchor="middle">Account</text> <text fill="currentColor" x="115.3" y="91" text-anchor="middle">your key pair</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="207" text-anchor="middle">DNS solver</text> <text fill="currentColor" x="115.3" y="223" text-anchor="middle">Cloudflare API</text></g> <g><rect stroke="currentColor" fill="none" x="282.79999999999995" y="54" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="364.09999999999997" y="75" text-anchor="middle">Let's Encrypt</text> <text fill="currentColor" x="364.09999999999997" y="91" text-anchor="middle">certificate authority</text></g> <g><rect stroke="currentColor" fill="none" x="282.79999999999995" y="186" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="364.09999999999997" y="207" text-anchor="middle">_acme-challenge</text> <text fill="currentColor" x="364.09999999999997" y="223" text-anchor="middle">TXT record</text></g></svg> <svg viewBox="0 0 230.6 580" style="max-width: 265px" role="img" aria-label="The authority signs only after it finds the TXT record in your zone, which only the domain's owner could publish."><defs><marker id="alc-bd-arrow-pgcah4" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="198.6" height="244" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="468" width="198.6" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="115.3" y1="102" x2="115.3" y2="346" marker-end="url(#alc-bd-arrow-pgcah4)"></line><text fill="currentColor" x="125.3" y="230">order</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="115.3" y1="250" x2="115.3" y2="494" marker-end="url(#alc-bd-arrow-pgcah4)"></line><text fill="currentColor" x="125.3" y="378">publish</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="115.3" y1="398" x2="115.3" y2="494" marker-end="url(#alc-bd-arrow-pgcah4)"></line><text fill="currentColor" x="125.3" y="452">checks</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">your deploy</text> <text fill="currentColor" x="28" y="487" text-anchor="start">example.com</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="75" text-anchor="middle">Account</text> <text fill="currentColor" x="115.3" y="91" text-anchor="middle">your key pair</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="223" text-anchor="middle">DNS solver</text> <text fill="currentColor" x="115.3" y="239" text-anchor="middle">Cloudflare API</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="371" text-anchor="middle">Let's Encrypt</text> <text fill="currentColor" x="115.3" y="387" text-anchor="middle">certificate authority</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="498" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="519" text-anchor="middle">_acme-challenge</text> <text fill="currentColor" x="115.3" y="535" text-anchor="middle">TXT record</text></g></svg>

The authority signs only after it finds the TXT record in your zone, which only the domain's owner could publish.

## Certificate authorities

A **certificate authority** (CA) is an organization browsers trust to vouch for domains. Let’s Encrypt and ZeroSSL issue certificates over ACME for free. Each has a **staging** environment, whose certificates browsers don’t trust but whose limits are generous, for testing, and a **production** one, with rate limits on how many certificates you can issue.

See [Certificate authorities](certificate-authorities.md).

## Accounts

You talk to a CA as an **account**: a key pair registered with it. Every order is signed with the account’s key. Some CAs, like ZeroSSL, also ask for **external account binding**, credentials that tie the ACME account to an account on their website.

```typescript
const account = yield* ACME.Account("Issuer", {
  ca: ACME.LetsEncryptStaging,
  contact: ["mailto:ops@example.com"],
  termsOfServiceAgreed: true,
});
```

## Orders and challenges

To get a certificate, the account places an **order** for one or more names. For each name, the CA sets a **challenge**: a task only the domain’s owner could complete. Once every challenge passes, the CA signs the certificate.

**DNS-01** is the challenge Alchemy uses: publish a TXT record at `_acme-challenge.example.com` with a value the CA gives you. It’s the only challenge that can prove control of a **wildcard** like `*.example.com`, and it needs no running server.

See [DNS validation](dns-validation.md).

## Certificates and keys

A **certificate** is a chain: your certificate, signed by the CA, and the intermediates linking it to a root browsers trust. It’s paired with a **private key**, which never leaves your hands. Whatever terminates TLS, a load balancer, a CDN or your own server, needs both.

```typescript
const cert = yield* ACME.Certificate("Wildcard", {
  account,
  identifiers: ["example.com", "*.example.com"],
  solver: Cloudflare.ACME.ZoneSolver({ zoneId: Config.String("CLOUDFLARE_ZONE_ID") }),
});
```

See [Using certificates](using-certificates.md).

## Renewal and revocation

Certificates expire, after 90 days for Let’s Encrypt, so they must be **renewed** before then. **Revocation** tells the CA to stop vouching for a certificate early, if its key leaked.

Renewal needs something to run. A certificate declared in your Stack checks its expiry on each deploy, so deploy on a schedule. One issued by your app at runtime renews when your app’s scheduler says.

See [Renewal & revocation](renewal.md).

## How @alchemy.run/acme models these concepts

An account and a certificate are resources, issued at deploy time and kept in your Stack’s state, with the DNS challenge answered by a solver for your DNS host. The certificate’s chain and key are outputs, passed to whatever serves TLS, on any provider. When an app must issue certificates itself, for its own customers’ domains, the `IssueCertificate` binding does the same at runtime. See [Runtime issuance](runtime.md).

Next, issue one: [Getting started](getting-started.md) deploys a staging certificate with a Cloudflare zone.
