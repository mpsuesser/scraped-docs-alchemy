---
url: https://alchemy.run/gcp/networking/vpc
title: "VPC & private networking"
description: "Give Cloud Run services Direct VPC egress to reach private IPs like Memorystore, and create your own VPC networks and subnets."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Most Google APIs, like Firestore, Pub/Sub, and Cloud Storage, are reached over their public endpoints, so a Cloud Run service needs no network setup. Some resources only have a private IP: Memorystore, AlloyDB, Filestore, and VMs without a public address. To reach those, give the service an interface on your VPC.

## A network spans regions

A VPC **network** on Google Cloud is global: one network reaches every region. Its **subnets** are regional, each with its own address range, and **firewall rules** on the network decide which traffic between them is allowed. A Cloud Run service joins the network through an interface in one subnet.

VpcVPC network · globalfirewall: allow 10.0.0.0/8

Privatesubnet · us-central1 · 10.0.0.0/24

Api's interfaceDirect VPC egress · 10.0.0.12

CacheMemorystore · 10.0.0.3

Europesubnet · europe-west1 · 10.1.0.0/24

worker-vmCompute Engine · 10.1.0.5

One network, a subnet in each region. The Api reaches the cache in its own subnet, and the VM in Europe over the same network, as the firewall allows.

## Direct VPC egress

Add `vpcAccess` to the service’s template:

```typescript
export default class Api extends GCP.Function<Api>()(
  "Api",
  {
    main: import.meta.url,
    template: {
      vpcAccess: {
        egress: "PRIVATE_RANGES_ONLY",
        networkInterfaces: [{ network: "default", subnetwork: "default" }],
      },
    },
  },
  Effect.gen(function* () {
    const redis = yield* GCP.Redis.ReadWriteRedis(Cache);
    return {
      fetch: redis.get("greeting").pipe(
        Effect.map((value) => HttpServerResponse.text(value ?? "hello")),
        Effect.orDie,
      ),
    };
  }).pipe(Effect.provide(GCP.Redis.ReadWriteRedisHttp)),
) {}
```

`PRIVATE_RANGES_ONLY` sends traffic for private addresses through the VPC. Everything else, including Google APIs, leaves the normal way. Use `ALL_TRAFFIC` to send everything through the VPC, for example to leave through a fixed NAT address.

## Create your own network

The `default` network works for most projects. Create a dedicated one when you need your own address ranges:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";

export const Network = GCP.Compute.Network("Vpc", {});

export const Subnet = Effect.gen(function* () {
  const network = yield* Network;
  return yield* GCP.Compute.Subnetwork("Private", {
    network: network.networkName,
    ipCidrRange: "10.0.0.0/24",
    privateIpGoogleAccess: true,
  });
});
```

New networks are in custom mode: you choose every subnet. Point the service at them:

```typescript
networkInterfaces: [
  { network: network.networkName, subnetwork: subnet.subnetworkName },
],
```

## Firewall rules

Allow traffic between your own ranges with a firewall rule:

```typescript
yield* GCP.Compute.Firewall("AllowInternal", {
  network: network.networkName,
  sourceRanges: ["10.0.0.0/8"],
  allowed: [{ protocol: "tcp" }],
});
```

## Where next

- [Memorystore](../data/memorystore.md) — Redis on a private IP.
- [Private services](private-services.md) — services that only other services can call.
- [Compute reference](https://alchemy.run/providers/gcp/compute#network) — networks, subnets, firewalls, and load balancers.
