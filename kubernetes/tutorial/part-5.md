---
url: https://alchemy.run/kubernetes/tutorial/part-5
title: "Part 5: Deploy to Your Own Cluster"
description: "Point the tutorial stack at a hosted Kubernetes cluster such as GKE, AKS, EKS, k3s, or anything in your kubeconfig. Keep the local cluster for development with stages, and clean up."
access_date: 2026-10-02T14:04:57.255Z
current_date: 2026-10-02T14:04:57.255Z
---

Everything you built in Parts 1–4 runs on a cluster on your machine. None of it is specific to that cluster. The Deployment, the Effect Jobs, and the Helm chart only need a `cluster` to run on. In this part you’ll point the stack at a cluster of your own, while keeping the local one for development.

## Choose a cluster

You need a cluster you can reach with `kubectl`. If you don’t have one, create it with your platform’s tools and follow its section of [Setup](../setup.md#connect-to-your-cluster):

- **GKE.** `gcloud container clusters get-credentials` writes the context. See [Setup → GKE](../setup.md#gke).
- **AKS.** `az aks get-credentials` writes the context. See [Setup → AKS](../setup.md#aks).
- **k3s, RKE2, DigitalOcean, and others.** Use the kubeconfig the platform gives you. See [Setup → k3s, RKE2, and other distributions](../setup.md#k3s-rke2-and-other-distributions).
- **Amazon EKS.** Create the cluster in this same Stack. See [Amazon EKS](../clusters/eks.md).

Then note its context name:

```sh
kubectl config get-contexts
```

The rest of this part uses a context named `prod`.

## Choose a registry

The smoke test and health check are built from your code, so they need a registry the cluster’s nodes can pull from. The local cluster came with one. For yours, pick a repository you can push to, such as `ghcr.io/<you>` on GitHub, and log in:

```sh
echo $GITHUB_TOKEN | docker login ghcr.io -u <you> --password-stdin
```

Make the pushed images readable by the cluster. Use a public package, or [pull credentials](../clusters/registries.md#let-the-nodes-pull) for a private one. See [Container registries](../clusters/registries.md).

## Pick the cluster by stage

Every Alchemy deploy targets a **stage**. By default that’s your own `live_<you>` stage, or you can name one with `--stage`. Keep the local cluster for your stage and use the hosted cluster for `prod`. In `src/infra.ts`, replace the `Cluster` declaration:

```typescript
import { Stage } from "alchemy";
import * as Kubernetes from "alchemy/Kubernetes";
import * as Effect from "effect/Effect";

export const Cluster = Kubernetes.LocalCluster("Cluster", {
  name: "alchemy",
});
export const Cluster = Effect.gen(function* () {
  const stage = yield* Stage;
  if (stage === "prod") {
    return Kubernetes.KubeConfig({
      context: "prod",
      registry: { server: "ghcr.io/you" },
    });
  }
  return yield* Kubernetes.LocalCluster("Cluster", { name: "alchemy" });
});
```

For `prod`, `Kubernetes.KubeConfig` connects through the `prod` context and pushes images to `ghcr.io/you`. Every other stage keeps the local cluster. Nothing else changes. The Namespace, Deployment, Jobs, and chart all take whichever cluster `Cluster` returns.

## Set the node architecture

Images are built for `amd64` nodes by default. If your cluster runs Arm nodes, such as AWS Graviton or Ampere, say so:

```typescript
return Kubernetes.KubeConfig({
  context: "prod",
  registry: { server: "ghcr.io/you" },
  architecture: "arm64",
});
```

## Stop returning the local context

`cluster.context` only exists on the local cluster. Remove it from the Stack’s outputs in `alchemy.run.ts`:

```typescript
return {
  context: cluster.context,
  namespace: web.namespace,
  service: web.serviceName,
  smokeTest: smokeTest.jobName,
};
```

## Deploy to prod

```sh
bun alchemy deploy --stage prod
```

```
Plan: 5 to create

+ HealthCheck (Kubernetes.Job)
+ MetricsServer (Kubernetes.HelmChart)
+ Namespace (Kubernetes.Manifest)
+ SmokeTest (Kubernetes.Job)
+ Web (Kubernetes.Deployment)

Proceed?
◉ Yes ○ No
Pushed ghcr.io/you/mycluster-smoketest-prod-…:7a31fba1…
Pushed ghcr.io/you/mycluster-healthcheck-prod-…:27bde98e…
✓ Namespace (Kubernetes.Manifest) created
✓ MetricsServer (Kubernetes.HelmChart) created
✓ Web (Kubernetes.Deployment) created
✓ SmokeTest (Kubernetes.Job) created
✓ HealthCheck (Kubernetes.Job) created
```

`prod` has its own state, so this deploy creates everything on the hosted cluster and leaves your local stage alone. There’s no `LocalCluster` in the plan, because the `prod` stage never declares one.

## Check the hosted cluster

The same commands work, with the `prod` context:

```sh
kubectl --context prod -n my-app get deployments,jobs,cronjobs
kubectl --context prod -n my-app logs job/<smokeTest>
```

## Skip what the platform already has

Many hosted clusters include metrics-server, and applying the chart over it would take ownership of its objects. Install the chart only where it’s missing:

```typescript
import { Stage } from "alchemy";
import * as Kubernetes from "alchemy/Kubernetes";
```

```typescript
yield* HealthCheck;

const metricsServer = yield* Kubernetes.HelmChart("MetricsServer", {
  cluster,
  // ...
});
const stage = yield* Stage;
if (stage !== "prod") {
  yield* Kubernetes.HelmChart("MetricsServer", {
    cluster,
    // ...
  });
}
```

Deploying `prod` again now plans the chart for deletion there.

## Clean up the local cluster

Destroy your own stage to remove everything on your machine, including the kind cluster and its registry:

```sh
bun alchemy destroy
```

Alchemy deletes the resources in reverse dependency order and the cluster last. The next `alchemy deploy` recreates it in about 30 seconds. Run `alchemy destroy --stage prod` when you want to remove the hosted deployment too.

## Recap

You started with an empty directory and now have:

- A stack that runs on a local cluster for development and on a hosted cluster in `prod`
- Effect programs built and pushed to whichever registry the cluster uses
- A Deployment, a Helm chart, and Jobs that don’t depend on where they run

## Where next

- **[Setup](../setup.md).** Connect to more kinds of clusters, check RBAC permissions, and deploy from CI.
- **[Container registries](../clusters/registries.md).** Push credentials, pull access, and node architecture.
- **[Deployments](../workloads/deployments.md).** Effect HTTP servers, Service types, and the pod template escape hatch.
- **[Jobs & CronJobs](../workloads/jobs.md).** Everything about image and Effect Jobs.
- **[Stages](../../environments/stages.md).** Per-developer, preview, and production stages.
- **[Amazon EKS](../clusters/eks.md).** Create the cluster in the same Stack and use AWS bindings.
