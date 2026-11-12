---
url: https://alchemy.run/aws/networking
title: "VPC & networking"
description: "How a VPC, its subnets, route tables, gateways and security groups decide what can reach what on AWS, which runtimes need one, and how to build one with the Network helper or by hand."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A **VPC** is a private network of your own inside an AWS region. Containers, virtual machines, databases and caches run inside one, and the VPC decides what can reach them: from the internet, from each other, and from AWS’s own services.

## Who needs a VPC

Serverless services don’t live in your VPC. A Lambda Function, a Bucket, a Table and a Queue are reached over AWS’s own endpoints, and a Function serves HTTP from its Function URL. An app made only of those needs no network at all.

A VPC comes in when something needs a network interface of its own: an [ECS](compute/ecs.md) task, an [EKS](compute/eks.md) node, an [EC2](compute/ec2.md) instance, an [Aurora](data/rds.md) database, a cache. A Lambda Function joins a VPC too when it has to reach one of those privately.

## Subnets split the network across zones

A VPC owns a range of private addresses, like `10.42.0.0/16`. **Subnets** divide it into smaller ranges, and each subnet sits in one Availability Zone. Put a subnet in each of at least two zones, so losing a data center leaves the app running in the other.

<svg viewBox="0 0 771 332" style="max-width: 887px" role="img" aria-label="Public subnets route through the internet gateway, so a load balancer there can be reached. Private subnets can't be reached, and call out through a NAT gateway in a public subnet."><defs><marker id="alc-dag-arrow-lv0xzm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="261" y="24" width="486" height="284" rx="10" ry="10"></rect><text fill="currentColor" x="273" y="41" text-anchor="start">VPC 10.42.0.0/16</text></g> <g><rect stroke="currentColor" fill="none" x="277" y="50" width="185" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="289" y="67" text-anchor="start">public a</text></g> <g><rect stroke="currentColor" fill="none" x="546" y="50" width="185" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="558" y="67" text-anchor="start">public b</text></g> <g><rect stroke="currentColor" fill="none" x="277" y="186" width="185" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="289" y="203" text-anchor="start">private a</text></g> <g><rect stroke="currentColor" fill="none" x="546" y="186" width="185" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="558" y="203" text-anchor="start">private b</text></g> <g><path d="M 177 108 L 271 108" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-lv0xzm)"></path></g><g><path d="M 638.5 212 L 638.5 146" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-lv0xzm)"></path><text fill="currentColor" x="646.5" y="175" text-anchor="start">out</text></g> <g transform="translate(24, 76)"><rect stroke="currentColor" fill="none" width="153" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="76.5" y="28" text-anchor="middle" dominant-baseline="middle">Internet gateway</text> <text fill="currentColor" x="76.5" y="46" text-anchor="middle" dominant-baseline="middle">in and out</text></g> <g transform="translate(293, 76)"><rect stroke="currentColor" fill="none" width="153" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="76.5" y="28" text-anchor="middle" dominant-baseline="middle">load balancer</text> <text fill="currentColor" x="76.5" y="46" text-anchor="middle" dominant-baseline="middle">reachable</text></g> <g transform="translate(562, 76)"><rect stroke="currentColor" fill="none" width="153" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="76.5" y="28" text-anchor="middle" dominant-baseline="middle">NAT gateway</text> <text fill="currentColor" x="76.5" y="46" text-anchor="middle" dominant-baseline="middle">out only</text></g> <g transform="translate(293, 212)"><rect stroke="currentColor" fill="none" width="153" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="76.5" y="28" text-anchor="middle" dominant-baseline="middle">database</text> <text fill="currentColor" x="76.5" y="46" text-anchor="middle" dominant-baseline="middle">no way in</text></g> <g transform="translate(562, 212)"><rect stroke="currentColor" fill="none" width="153" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="76.5" y="28" text-anchor="middle" dominant-baseline="middle">containers</text> <text fill="currentColor" x="76.5" y="46" text-anchor="middle" dominant-baseline="middle">call out</text></g></svg> <svg viewBox="0 0 417 388" style="max-width: 480px" role="img" aria-label="Public subnets route through the internet gateway, so a load balancer there can be reached. Private subnets can't be reached, and call out through a NAT gateway in a public subnet."><defs><marker id="alc-dag-arrow-52ayu6" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="10" y="94" width="397" height="284" rx="10" ry="10"></rect><text fill="currentColor" x="124.5" y="111" text-anchor="start">VPC 10.42.0.0/16</text></g> <g><rect stroke="currentColor" fill="none" x="26" y="120" width="177" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="38" y="137" text-anchor="start">public a</text></g> <g><rect stroke="currentColor" fill="none" x="26" y="256" width="177" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="38" y="273" text-anchor="start">public b</text></g> <g><rect stroke="currentColor" fill="none" x="214" y="120" width="177" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="226" y="137" text-anchor="start">private a</text></g> <g><rect stroke="currentColor" fill="none" x="214" y="256" width="177" height="106" rx="10" ry="10" stroke-dasharray="5 4"></rect><text fill="currentColor" x="226" y="273" text-anchor="start">private b</text></g> <g><path d="M 114.5 74 L 114.5 114" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-52ayu6)"></path></g><g><path d="M 230 314 L 193 314" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-52ayu6)"></path><text fill="currentColor" x="211.5" y="308" text-anchor="middle">out</text></g> <g transform="translate(42, 10)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">Internet gateway</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">in and out</text></g> <g transform="translate(42, 146)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">load balancer</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">reachable</text></g> <g transform="translate(42, 282)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">NAT gateway</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">out only</text></g> <g transform="translate(230, 146)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">database</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">no way in</text></g> <g transform="translate(230, 282)"><rect stroke="currentColor" fill="none" width="145" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="72.5" y="28" text-anchor="middle" dominant-baseline="middle">containers</text> <text fill="currentColor" x="72.5" y="46" text-anchor="middle" dominant-baseline="middle">call out</text></g></svg>

Public subnets route through the internet gateway, so a load balancer there can be reached. Private subnets can't be reached, and call out through a NAT gateway in a public subnet.

What makes a subnet public or private is its **route table**. A public subnet routes internet traffic to an **internet gateway**, so things in it can have public addresses and be reached from outside. A private subnet has no route in from the internet. If it routes outbound traffic to a **NAT gateway** in a public subnet, things in it can call out, to download packages or call an API, and still can’t be reached. Load balancers go in public subnets. Databases, and usually containers, go in private ones.

## Security groups are firewalls around each resource

A **security group** is a set of rules around each network interface: which ports accept traffic, and from where. It’s stateful, so the reply to an allowed request is always allowed back.

A rule can name another security group instead of an address. “Port 5432, from anything in the `App` group” lets your containers reach the database however many there are and whatever addresses they get:

```typescript
const database = yield* AWS.EC2.SecurityGroup("Database", {
  vpcId: network.vpcId,
  ingress: [{ ipProtocol: "tcp", fromPort: 5432, toPort: 5432, referencedGroupId: app.groupId }],
});
```

## Build the usual layout

`AWS.EC2.Network` builds a VPC with an internet gateway, one public and one private subnet in each of two zones, and the route tables between them:

```typescript
import * as AWS from "@alchemy.run/aws";

export const AppNetwork = AWS.EC2.Network("AppNetwork", { cidrBlock: "10.42.0.0/16" });
```

The subnet ranges are worked out from the VPC’s. Pass `vpcId`, `publicSubnetIds` and `privateSubnetIds` from the result to whatever runs in it:

```typescript
const network = yield* AppNetwork;

yield* AWS.ECS.Service("Api", {
  cluster,
  task: apiTask,
  vpcId: network.vpcId,
  subnets: network.publicSubnetIds,
  assignPublicIp: true,
  loadBalancer: true,
});
```

The tasks and their load balancer share these subnets. Public subnets with public addresses let the tasks pull their image without a NAT gateway. To keep the tasks private, add NAT, below, and put them in `privateSubnetIds`.

## Let private subnets reach out

Private subnets are isolated by default. Add NAT when what runs in them needs the internet:

```typescript
export const AppNetwork = AWS.EC2.Network("AppNetwork", {
  cidrBlock: "10.42.0.0/16",
  nat: "single",
  gatewayEndpoints: ["s3", "dynamodb"],
});
```

`"single"` shares one NAT gateway between the zones. `"per-az"` puts one in each, so losing a zone doesn’t cut the others off, at the cost of a gateway each. NAT gateways are billed by the hour and by the gigabyte.

**Gateway endpoints** route traffic for S3 and DynamoDB over AWS’s network instead, for free. With them, a private subnet reaches its Buckets and Tables without NAT at all.

## Build it by hand

`Network` returns every resource it creates, so you can keep building on it. When you need a different layout, declare the pieces yourself. A VPC and its internet gateway:

```typescript
const vpc = yield* AWS.EC2.Vpc("AppVpc", {
  cidrBlock: "10.80.0.0/16",
  enableDnsSupport: true,
  enableDnsHostnames: true,
});
const igw = yield* AWS.EC2.InternetGateway("AppIgw", { vpcId: vpc.vpcId });
```

A subnet in a zone:

```typescript
const publicA = yield* AWS.EC2.Subnet("PublicA", {
  vpcId: vpc.vpcId,
  cidrBlock: "10.80.1.0/24",
  availabilityZone: "us-east-1a",
  mapPublicIpOnLaunch: true,
});
```

A route table that makes it public:

```typescript
const publicRoutes = yield* AWS.EC2.RouteTable("PublicRoutes", { vpcId: vpc.vpcId });

yield* AWS.EC2.Route("PublicInternet", {
  routeTableId: publicRoutes.routeTableId,
  destinationCidrBlock: "0.0.0.0/0",
  gatewayId: igw.internetGatewayId,
});

yield* AWS.EC2.RouteTableAssociation("PublicA", {
  routeTableId: publicRoutes.routeTableId,
  subnetId: publicA.subnetId,
});
```

And a NAT gateway for the private subnets, with a static address, in a public subnet:

```typescript
const eip = yield* AWS.EC2.EIP("NatEip", { domain: "vpc" });
const nat = yield* AWS.EC2.NatGateway("Nat", {
  subnetId: publicA.subnetId,
  allocationId: eip.allocationId,
  connectivityType: "public",
});

yield* AWS.EC2.Route("PrivateInternet", {
  routeTableId: privateRoutes.routeTableId,
  destinationCidrBlock: "0.0.0.0/0",
  natGatewayId: nat.natGatewayId,
});
```

Network ACLs, peering, interface endpoints and IPv6 gateways are in the [EC2 reference](https://alchemy.run/providers/aws/ec2#vpc).
