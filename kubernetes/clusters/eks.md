---
url: https://alchemy.run/kubernetes/clusters/eks
title: "Amazon EKS"
description: "Run Alchemy's Kubernetes resources on Amazon EKS. Find the guide for creating clusters, ECR images, Pod Identity, and load balancers."
access_date: 2026-10-02T14:04:57.255Z
current_date: 2026-10-02T14:04:57.255Z
---

Everything in these Kubernetes docs works on
[Amazon EKS](https://aws.amazon.com/eks/). The EKS guide lives with
the rest of the AWS docs:

**[EKS guide →](../../aws/compute/eks.md)**

It covers creating an Auto Mode cluster in the same Stack, granting
access, and what EKS adds when you pass the cluster as the `cluster`
of a `Kubernetes.*` resource:

- **No kubeconfig.** Alchemy authenticates with your AWS
  credentials.
- **A registry per workload.** `main` and `context` workloads are
  built into ECR, so no `registry` setting is needed.
- **AWS bindings.** Bindings such as `AWS.DynamoDB.PutItem(table)`
  grant IAM permissions to the workload through EKS Pod Identity.
- **Public load balancers.** `LoadBalancer` Services get an
  internet-facing Network Load Balancer.

To get started with AWS, see [AWS setup](../../aws/setup.md).

## Use an EKS cluster through kubeconfig

A context written by `aws eks update-kubeconfig` works with
[`Kubernetes.KubeConfig`](connecting.md) like any
other cluster. That path only authenticates. Workloads need a
[registry](registries.md) for `main` and `context`
images, and AWS bindings aren't wired to Pod Identity. Pass the
`AWS.EKS.Cluster` resource, or an `aws-eks` connection, to get the
full integration.
