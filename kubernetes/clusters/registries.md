---
url: https://alchemy.run/kubernetes/clusters/registries
title: "Container registries"
description: "Give a Kubernetes connection a registry so Deployments and Jobs built from Effect programs or Dockerfiles are pushed where the cluster's nodes can pull them, such as GHCR, Docker Hub, Artifact Registry, ACR, or a registry of your own."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A Kubernetes node can only run an image it can pull from a registry. Workloads that use a pre-built `image` need nothing extra, but a `main` Effect program or a `context` Dockerfile is built on your machine and has to be pushed somewhere first. The connection’s `registry` says where:

```typescript
const cluster = Kubernetes.KubeConfig({
  context: "prod-us-east",
  registry: { server: "ghcr.io/acme" },
});

const worker = yield* Kubernetes.Job(
  "Worker",
  { cluster, main: import.meta.url },
  Effect.gen(function* () {
    return { run: Effect.log("working") };
  }),
);
```

<svg viewBox="0 0 628.5999999999999 136" style="max-width: 723px" role="img" aria-label="The image is built where you deploy and pulled by the node that runs it, so the registry has to be one both can reach."><defs><marker id="alc-bd-arrow-0f0w3a" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="172.2" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="440.4" y="24" width="172.19999999999993" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="170.2" y1="78" x2="242.2" y2="78" marker-end="url(#alc-bd-arrow-0f0w3a)"></line><text fill="currentColor" x="208.2" y="70" text-anchor="middle">push</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="382.4" y1="78" x2="454.4" y2="78" marker-end="url(#alc-bd-arrow-0f0w3a)"></line><text fill="currentColor" x="420.4" y="70" text-anchor="middle">pull</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">your machine</text> <text fill="currentColor" x="452.4" y="43" text-anchor="start">cluster</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="75" text-anchor="middle">alchemy deploy</text> <text fill="currentColor" x="102.1" y="91" text-anchor="middle">builds api:&lt;hash&gt;</text></g> <g><rect stroke="currentColor" fill="none" x="246.2" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="314.29999999999995" y="75" text-anchor="middle">ghcr.io/acme</text> <text fill="currentColor" x="314.29999999999995" y="91" text-anchor="middle">registry</text></g> <g><rect stroke="currentColor" fill="none" x="458.4" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="526.5" y="75" text-anchor="middle">node</text> <text fill="currentColor" x="526.5" y="91" text-anchor="middle">pulls api:&lt;hash&gt;</text></g></svg> <svg viewBox="0 0 204.2 432" style="max-width: 235px" role="img" aria-label="The image is built where you deploy and pulled by the node that runs it, so the registry has to be one both can reach."><defs><marker id="alc-bd-arrow-95e7x2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="172.2" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="320" width="172.2" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="102.1" y1="102" x2="102.1" y2="198" marker-end="url(#alc-bd-arrow-95e7x2)"></line><text fill="currentColor" x="112.1" y="156">push</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="102.1" y1="250" x2="102.1" y2="346" marker-end="url(#alc-bd-arrow-95e7x2)"></line><text fill="currentColor" x="112.1" y="304">pull</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">your machine</text> <text fill="currentColor" x="28" y="339" text-anchor="start">cluster</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="75" text-anchor="middle">alchemy deploy</text> <text fill="currentColor" x="102.1" y="91" text-anchor="middle">builds api:&lt;hash&gt;</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="223" text-anchor="middle">ghcr.io/acme</text> <text fill="currentColor" x="102.1" y="239" text-anchor="middle">registry</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="136.2" height="48" rx="8"></rect><text fill="currentColor" x="102.1" y="371" text-anchor="middle">node</text> <text fill="currentColor" x="102.1" y="387" text-anchor="middle">pulls api:&lt;hash&gt;</text></g></svg>

The image is built where you deploy and pulled by the node that runs it, so the registry has to be one both can reach.

On deploy, Alchemy bundles the program, builds the image with your local Docker, pushes it to `ghcr.io/acme/<name>:<hash>`, and points the Job at that reference.

Two kinds of cluster fill in the registry for you:

- [`Kubernetes.LocalCluster`](local.md) includes a local registry.
- [Amazon EKS](eks.md) pushes each workload to its own ECR repository.

## Choose the server

`server` is the registry host, optionally followed by the namespace images go under:

- **GitHub Container Registry.** Use `ghcr.io/<owner>`.
- **Docker Hub.** Use `docker.io/<user>`.
- **Google Artifact Registry.** Use `<region>-docker.pkg.dev/<project>/<repository>`.
- **Azure Container Registry.** Use `<name>.azurecr.io`.
- **A registry you run.** Use its host and port, for example `registry.internal:5000`.

Each workload pushes to its own repository under `server`, named after the stack, stage, and logical ID, and tags every image with a hash of its content.

## Authenticate pushes

Without credentials, Alchemy pushes with your machine’s own Docker login, so any registry you’ve run `docker login` against works:

```sh
echo $GITHUB_TOKEN | docker login ghcr.io -u acme-bot --password-stdin
```

For CI, you can keep the credential in the program instead. Pass `username` and `password`:

```typescript
import * as Config from "effect/Config";

const token = yield* Config.Redacted("GHCR_TOKEN");

const cluster = Kubernetes.KubeConfig({
  context: "prod-us-east",
  registry: { server: "ghcr.io/acme", username: "acme-bot", password: token },
});
```

The connection is stored in Alchemy’s state with each resource. A `Redacted` password is stored as a redacted value, but anyone who can read the state can read it, so prefer `docker login` on machines you control.

## Let the nodes pull

Pushing and pulling are separate. The nodes need read access to the repository. Public repositories need nothing. For private ones, use your platform’s node credentials. GKE and AKS can grant the nodes access to Artifact Registry and ACR. You can also use an image-pull Secret referenced from the pod template:

```typescript
const pullSecret = yield* Kubernetes.Manifest("GhcrPullSecret", {
  cluster,
  manifest: {
    apiVersion: "v1",
    kind: "Secret",
    type: "kubernetes.io/dockerconfigjson",
    metadata: { name: "ghcr", namespace: "apps" },
    data: { ".dockerconfigjson": dockerConfigJsonBase64 },
  },
});

const worker = yield* Kubernetes.Job(
  "Worker",
  {
    cluster,
    main: import.meta.url,
    namespace: "apps",
    podTemplate: {
      spec: { imagePullSecrets: [{ name: pullSecret.name }] },
    },
  },
  Effect.gen(function* () {
    return { run: Effect.log("working") };
  }),
);
```

## Match the node architecture

Images are built for `linux/amd64` unless you say otherwise. For Arm nodes, set `architecture` on the connection so every workload builds for them:

```typescript
const cluster = Kubernetes.KubeConfig({
  context: "graviton",
  registry: { server: "ghcr.io/acme" },
  architecture: "arm64",
});
```

A workload’s own `architecture` prop overrides the connection’s. `LocalCluster` sets the architecture to match your machine.

## Old images

Alchemy pushes a new tag whenever a workload’s image changes and never deletes tags from registries you configure. Use your registry’s retention or cleanup policies to prune old images.

## Where next

- **[Container images](../workloads/images.md).** The `image`, `context`, and `main` sources.
- **[Connecting to clusters](connecting.md).** Every connection form that accepts a `registry`.
