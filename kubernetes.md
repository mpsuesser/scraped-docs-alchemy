---
url: https://alchemy.run/kubernetes
title: "Kubernetes"
description: "Run containers and Effect programs on any Kubernetes cluster with Alchemy. Declare Deployments, Jobs, raw manifests, and Helm charts in the same typed program as the rest of your infrastructure. No YAML, no kubectl apply."
access_date: 2026-10-02T14:04:57.255Z
current_date: 2026-10-02T14:04:57.255Z
---

`alchemy/Kubernetes` manages workloads on **any Kubernetes cluster**, such as a cluster on your laptop, k3s on a VM, an on-prem fleet, GKE, AKS, or EKS. You describe servers, jobs, and raw Kubernetes objects as Resources in TypeScript. Alchemy builds your programs into images and applies everything to the cluster with server-side apply. It tracks everything in state and deletes it on destroy.

```typescript
import * as Alchemy from "alchemy";
import * as Kubernetes from "alchemy/Kubernetes";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Kubernetes.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const cluster = yield* Kubernetes.LocalCluster("Cluster", {
      name: "alchemy",
    });

    const web = yield* Kubernetes.Deployment("Web", {
      cluster,
      name: "web",
      image: "ghcr.io/stefanprodan/podinfo:6.15.0",
      port: 9898,
      replicas: 2,
      serviceType: "ClusterIP",
    });

    return { service: web.serviceName };
  }),
);
```

`Kubernetes.LocalCluster` starts a cluster on your machine, so you can try everything here without a hosted cluster. When you’re ready, point the same resources at a real one.

New here? Follow [Setup](kubernetes/setup.md), then start the [tutorial](kubernetes/tutorial/part-1.md).

## Clusters

Every resource takes a `cluster` prop that says where it runs, how to authenticate, and where to push images.

- **[Connecting to clusters](kubernetes/clusters/connecting.md).** Use `Kubernetes.KubeConfig(...)` for anything `kubectl` can reach, or a raw `Kubernetes.Connection` with a bearer token, client certificate, or exec plugin.
- **[Container registries](kubernetes/clusters/registries.md).** Workloads built from your code are pushed to a registry so the cluster can pull them.
- **[Local clusters](kubernetes/clusters/local.md).** `Kubernetes.LocalCluster` runs a [kind](https://kind.sigs.k8s.io/) cluster in Docker with an image registry built in.
- **[Amazon EKS](kubernetes/clusters/eks.md).** Pass an EKS cluster from the same Stack and get ECR images, AWS bindings, and public load balancers.
- **[Cluster adapters](kubernetes/clusters/cluster-adapters.md).** An adapter is the extension point that gives each cluster platform its authentication, registry, and identity behavior.

## Workloads

- **[Deployments](kubernetes/workloads/deployments.md).** A replicated server made of a Kubernetes `Deployment`, `Service`, and `ServiceAccount`. It runs an image or an Effect HTTP server.
- **[Jobs & CronJobs](kubernetes/workloads/jobs.md).** Run-to-completion work, from a container image or an Effect program. Set `schedule` and the Job becomes a `CronJob`.
- **[Container images](kubernetes/workloads/images.md).** Every workload runs exactly one of `image` (a registry reference), `context` (your Dockerfile), or `main` (a bundled Effect program).
- **[Configuration & bindings](kubernetes/workloads/bindings.md).** Environment variables, values from other resources, and bindings.
- **[How objects are managed](kubernetes/workloads/object-lifecycle.md).** Server-side apply, apply and delete ordering, pruning, drift, and what triggers a replacement.

## Objects

- **[Manifests](kubernetes/objects/manifests.md).** Apply any single Kubernetes object, such as Namespaces, ConfigMaps, StatefulSets, Ingresses, and custom resources.
- **[Helm charts](kubernetes/objects/helm-charts.md).** Render a chart with the local `helm` CLI and apply its objects as one resource.

## What are you building?

- **An HTTP service from a public image.** Use [`Deployment`](kubernetes/workloads/deployments.md) with `image`.
- **An HTTP server written in Effect.** Use [`Deployment`](kubernetes/workloads/deployments.md#effect-servers) with `main`.
- **A database migration, seed, or smoke test.** Use [`Job`](kubernetes/workloads/jobs.md).
- **A nightly batch task.** Use [`Job`](kubernetes/workloads/jobs.md#run-on-a-schedule) with `schedule`.
- **A Namespace, ConfigMap, or Secret.** Use [`Manifest`](kubernetes/objects/manifests.md).
- **A StatefulSet, Ingress, or custom resource.** Use [`Manifest`](kubernetes/objects/manifests.md).
- **A cluster add-on (metrics, ingress, operators).** Use [`HelmChart`](kubernetes/objects/helm-charts.md).

## Where next

- [Setup](kubernetes/setup.md) shows how to install Alchemy and connect it to a local or hosted cluster.
- [Tutorial](kubernetes/tutorial/part-1.md) takes you from an empty directory to a local cluster running a service, Effect Jobs, a CronJob, and a Helm chart. It ends on a cluster of your own.
- API reference for [`LocalCluster`](https://alchemy.run/providers/kubernetes/reference/localcluster#localcluster), [`Deployment`](https://alchemy.run/providers/kubernetes/reference/workloads#deployment), [`Job`](https://alchemy.run/providers/kubernetes/reference/workloads#job), [`Manifest`](https://alchemy.run/providers/kubernetes/reference/manifest#manifest), [`HelmChart`](https://alchemy.run/providers/kubernetes/reference/helm#helmchart).
