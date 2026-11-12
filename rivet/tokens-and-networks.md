---
url: https://alchemy.run/rivet/tokens-and-networks
title: "Tokens and networks"
description: "Who can reach a Rivet control plane and with what — the admin token and everything that holds it, the HTTPS address each control plane gets without a domain, nodes that only connect out, and the credentials a deploy needs in CI."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A control plane is reached two ways. Callers send calls to its **gateway**, and nodes connect to it to take work. Both carry the control plane’s **admin token**, and both reach it over HTTPS.

## The admin token

A control plane you run gets an admin token when it’s created: 32 random bytes, kept in your state. The engines check every request against it.

Everything that talks to the control plane holds it:

- each node, as `RIVET_TOKEN`,
- each runtime that calls a class or a stream, as a secret environment value bound when it deploys,
- each deploy, which creates namespaces and pools with it.

The token can do anything on the control plane, in every namespace. Keep a runtime’s environment as secret as the token itself. Calls send it in the `x-rivet-token` header, never in the URL, so it stays out of proxy and load balancer logs.

## An HTTPS address without a domain

Every control plane you run serves its gateway over HTTPS, at an address its cloud provides. You don’t need a domain of your own.

On AWS, `RivetAWS.Ec2ControlPlane` and `EcsControlPlane` answer at `https://….cloudfront.net`: CloudFront reaches an internal load balancer through a VPC origin.

On GCP, `RivetGCP.ComputeEngineControlPlane` answers at `https://….run.app`: a proxy on Cloud Run reaches the engines through a Serverless VPC Access connector.

On Hetzner, `RivetHetzner.ServerControlPlane` answers at an address like `https://203-0-113-7.sslip.io`: a Caddy server with a Let’s Encrypt certificate sits in front of a private load balancer.

On Fly, `RivetFly.MachineControlPlane` answers at `https://….fly.dev`, through Fly’s proxy. `RivetDocker.SwarmControlPlane` runs on your machine, at `http://localhost:6420`.

The engines themselves have no public address. Only the HTTPS front reaches them, over the control plane’s private network.

<svg viewBox="0 0 457.8 520" style="max-width: 526px" role="img" aria-label="On AWS, callers reach the engines only through CloudFront: nothing in the VPC has a public address. Nodes inside the VPC use the internal load balancer directly."><defs><marker id="alc-bd-arrow-s01tod" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="276" width="425.8" height="228" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="64" x2="100" y2="144" marker-end="url(#alc-bd-arrow-s01tod)"></line><text fill="currentColor" x="110" y="110">x-rivet-token</text></g> <g><g><rect stroke="currentColor" fill="none" x="78" y="273" width="44" height="6"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="78" y1="276" x2="122" y2="276"></line></g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="196" x2="100" y2="302" marker-end="url(#alc-bd-arrow-s01tod)"></line><text fill="currentColor" x="110" y="255">VPC origin</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="330" x2="287.8" y2="330" marker-end="url(#alc-bd-arrow-s01tod)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="486" x2="100" y2="302" marker-end="url(#alc-bd-arrow-s01tod)"></line><text fill="currentColor" x="110" y="400">connects out</text></g> <text fill="currentColor" x="429.8" y="295" text-anchor="end">Control plane VPC</text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="37" text-anchor="middle">Api</text> <text fill="currentColor" x="100" y="53" text-anchor="middle">Lambda</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="148" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="169" text-anchor="middle">CloudFront</text> <text fill="currentColor" x="100" y="185" text-anchor="middle">HTTPS</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="306" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="327" text-anchor="middle">Load balancer</text> <text fill="currentColor" x="100" y="343" text-anchor="middle">internal</text></g> <g><rect stroke="currentColor" fill="none" x="291.8" y="306" width="132" height="48" rx="8"></rect><text fill="currentColor" x="357.8" y="327" text-anchor="middle">Engines</text> <text fill="currentColor" x="357.8" y="343" text-anchor="middle">3 instances</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="438" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="459" text-anchor="middle">Nodes</text> <text fill="currentColor" x="100" y="475" text-anchor="middle">pool Default</text></g></svg> <svg viewBox="0 0 211.8 700" style="max-width: 244px" role="img" aria-label="On AWS, callers reach the engines only through CloudFront: nothing in the VPC has a public address. Nodes inside the VPC use the internal load balancer directly."><defs><marker id="alc-bd-arrow-ub6k12" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="292" width="168" height="392" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="64" x2="100" y2="160" marker-end="url(#alc-bd-arrow-ub6k12)"></line><text fill="currentColor" x="110" y="118">x-rivet-token</text></g> <g><g><rect stroke="currentColor" fill="none" x="78" y="289" width="44" height="6"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="78" y1="292" x2="122" y2="292"></line></g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="212" x2="100" y2="318" marker-end="url(#alc-bd-arrow-ub6k12)"></line><text fill="currentColor" x="110" y="271">VPC origin</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="370" x2="100" y2="614" marker-end="url(#alc-bd-arrow-ub6k12)"></line></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="518" x2="100" y2="318" marker-end="url(#alc-bd-arrow-ub6k12)"></line><text fill="currentColor" x="110" y="424">connects out</text></g> <text fill="currentColor" x="172" y="311" text-anchor="end">Control plane VPC</text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="37" text-anchor="middle">Api</text> <text fill="currentColor" x="100" y="53" text-anchor="middle">Lambda</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="185" text-anchor="middle">CloudFront</text> <text fill="currentColor" x="100" y="201" text-anchor="middle">HTTPS</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="322" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="343" text-anchor="middle">Load balancer</text> <text fill="currentColor" x="100" y="359" text-anchor="middle">internal</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="618" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="639" text-anchor="middle">Engines</text> <text fill="currentColor" x="100" y="655" text-anchor="middle">3 instances</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="470" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="491" text-anchor="middle">Nodes</text> <text fill="currentColor" x="100" y="507" text-anchor="middle">pool Default</text></g></svg>

On AWS, callers reach the engines only through CloudFront: nothing in the VPC has a public address. Nodes inside the VPC use the internal load balancer directly.

## Nodes only connect out

Nodes open a connection to the control plane and wait for work on it, so nothing connects in to them. A pool needs no public address, no load balancer and no inbound firewall rule.

A node inside the control plane’s own network, like an `EcsPool` in the VPC of an `Ec2ControlPlane`, connects to the private address. A node anywhere else, like a `RivetFly.MachinePool` for that same control plane, connects to the HTTPS address.

## Rivet Services stays private

A [Rivet Services](durable-streams.md) node also serves Durable Streams on a port of its own, and that port checks no token. Services pools publish no port, and callers reach every stream through the gateway, with the token. Don’t open that port to anyone.

## In CI

A deploy needs the credentials of the cloud the control plane runs on, read the way that provider reads them in CI: `AWS_*` for AWS, `HCLOUD_TOKEN` for Hetzner, and so on. A control plane you run creates its own admin token, so it needs nothing from Rivet.

A Rivet Cloud control plane needs a cluster token from the Rivet dashboard and the cluster’s name. `RivetCloud.ControlPlane` reads them from the environment:

```sh
RIVET_CLOUD_TOKEN=...
RIVET_CLOUD_CLUSTER=my-cluster
```

See [Sign in from CI](rivet-cloud.md#sign-in-from-ci).

See [CI](../environments/ci.md) and [Secrets](secrets.md).
