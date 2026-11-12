---
url: https://alchemy.run/gcp/compute/choosing-a-runtime
title: "Choosing a runtime"
description: "How Cloud Run services, jobs and worker pools, Cloud Functions and GKE differ in what triggers them, how they scale and what they cost, and how to choose one for each part of an app."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Google Cloud gives you several places to run code. Most of them are Cloud Run underneath: containers that Google starts, scales and stops for you. They differ in what starts them and how long they live. Every one runs the same kind of Alchemy program, takes the same bindings, and runs as a service account of its own.

FunctionGoogle builds

Serviceon requests

Jobto completion

Worker poolalways on

GKEAutopilot

your program

you

you

you

you

you

image

Google

you

you

you

you

scaling

Google

Google

you

you

you

orchestration

Google

Google

Google

Google

you

OS & patching

Google

Google

Google

Google

Google

hardware

Google

Google

Google

Google

Google

From left to right, Google manages less and you decide more. None of them leaves the operating system to you.

## The same program on every runtime

Whichever you pick, you declare it with an id, its configuration and an Effect. Construction binds what it uses. A runtime that answers HTTP returns `fetch`, and one that runs to completion returns `run`:

```typescript
export default class Reindex extends GCP.Run.Job<Reindex>()(
  "Reindex",
  { main: import.meta.url },
  Effect.gen(function* () {
    const db = yield* GCP.Firestore.ReadDatabase(Links);
    return {
      run: Effect.gen(function* () {
        const { documents } = yield* db.list("links");
        yield* Effect.log(\`reindexing ${documents.length} links\`);
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Firestore.ReadDatabaseHttp)),
) {}
```

So moving part of an app from one runtime to another changes its declaration, not its bindings. Choose by how the work behaves:

## Cloud Run services: answer HTTP

A [Cloud Run service](cloud-run.md) runs your `fetch` handler in containers that start as requests arrive, serve many requests each, and stop when traffic does. An idle service costs nothing. Pub/Sub, Cloud Storage, Cloud Scheduler and Eventarc can all push to it, so it handles events as well as requests.

It’s an easy and cheap place to start, and it fits APIs, websites and event handlers.

<svg viewBox="0 0 698.4 140" style="max-width: 803px" role="img" aria-label="An instance takes many requests at once, and Cloud Run stops every instance when traffic does."><line stroke-opacity="0.2" stroke="currentColor" x1="462.4" y1="24" x2="462.4" y2="134"></line><g><rect stroke="currentColor" fill="none" x="118.4" y="33" width="576" height="98" rx="8"></rect><text fill="currentColor" x="4" y="80">instance 1</text> <text fill="currentColor" x="4" y="94">up to 80 at once</text> <g><rect stroke="currentColor" fill="none" x="126.4" y="40" width="39.19999999999999" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="165.6" y="40" width="140.00000000000003" height="24" rx="5"></rect><text fill="currentColor" x="235.60000000000002" y="56" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="204.8" y="70" width="134.40000000000003" height="24" rx="5"></rect><text fill="currentColor" x="272" y="86" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="238.4" y="100" width="145.6" height="24" rx="5"></rect><text fill="currentColor" x="311.2" y="116" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="462.4" y="18" text-anchor="middle">idle: scaled to zero</text></svg> <svg viewBox="0 0 330 150" style="max-width: 379px" role="img" aria-label="An instance takes many requests at once, and Cloud Run stops every instance when traffic does."><line stroke-opacity="0.2" stroke="currentColor" x1="190.79999999999998" y1="24" x2="190.79999999999998" y2="144"></line><g><rect stroke="currentColor" fill="none" x="0" y="51" width="326" height="90" rx="8"></rect><text fill="currentColor" x="0" y="44">instance 1 <tspan fill="currentColor" dx="8">up to 80 at once</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="22.259999999999998" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="22.259999999999998" y="54" width="79.5" height="24" rx="5"></rect><text fill="currentColor" x="62.01" y="70" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="44.519999999999996" y="84" width="76.32000000000001" height="24" rx="5"></rect><text fill="currentColor" x="82.68" y="100" text-anchor="middle">request</text></g> <g><rect stroke="currentColor" fill="none" x="63.6" y="114" width="82.68" height="24" rx="5"></rect><text fill="currentColor" x="104.94" y="130" text-anchor="middle">request</text></g></g> <text fill="currentColor" x="190.79999999999998" y="18" text-anchor="middle">idle: scaled to zero</text></svg>

An instance takes many requests at once, and Cloud Run stops every instance when traffic does.

## Cloud Run jobs: run to completion

A [job](jobs.md) starts, does its work and exits. Nothing runs, or is billed, between executions. Start one from another runtime with the `RunJob` binding, or on a schedule with Cloud Scheduler.

It fits batch work: reindexing, exports, migrations, nightly reports.

<svg viewBox="0 0 640.4 124" style="max-width: 736px" role="img" aria-label="An execution runs its tasks to completion and exits. Between executions nothing runs and nothing is billed."><line stroke-opacity="0.2" stroke="currentColor" x1="124.4" y1="24" x2="124.4" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="460.4" y1="24" x2="460.4" y2="118"></line><g><text fill="currentColor" x="4" y="56">task 0</text> <g><rect stroke="currentColor" fill="none" x="124.4" y="40" width="190.40000000000006" height="24" rx="5"></rect><text fill="currentColor" x="219.60000000000002" y="56" text-anchor="middle">runs, then exits</text></g> <g><rect stroke="currentColor" fill="none" x="460.4" y="40" width="134.39999999999998" height="24" rx="5"></rect><text fill="currentColor" x="527.5999999999999" y="56" text-anchor="middle">next execution</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="640.4" y2="74"></line><text fill="currentColor" x="4" y="100">task 1</text> <g><rect stroke="currentColor" fill="none" x="124.4" y="84" width="145.6" height="24" rx="5"></rect><text fill="currentColor" x="197.2" y="100" text-anchor="middle">runs, then exits</text></g></g> <text fill="currentColor" x="124.4" y="18" text-anchor="middle">RunJob</text> <text fill="currentColor" x="460.4" y="18" text-anchor="middle">03:00</text></svg> <svg viewBox="0 0 330 159" style="max-width: 379px" role="img" aria-label="An execution runs its tasks to completion and exits. Between executions nothing runs and nothing is billed."><line stroke-opacity="0.2" stroke="currentColor" x1="31.8" y1="24" x2="31.8" y2="153"></line><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="24" x2="222.6" y2="153"></line><g><text fill="currentColor" x="0" y="44">task 0</text> <g><rect stroke="currentColor" fill="none" x="31.8" y="54" width="108.12000000000002" height="24" rx="5"></rect><text fill="currentColor" x="31.8" y="90" text-anchor="start">runs, then exits</text></g> <g><rect stroke="currentColor" fill="none" x="222.6" y="54" width="76.32000000000002" height="24" rx="5"></rect><text fill="currentColor" x="222.6" y="90" text-anchor="start">next execution</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="99" x2="330" y2="99"></line><text fill="currentColor" x="0" y="113">task 1</text> <g><rect stroke="currentColor" fill="none" x="31.8" y="123" width="82.67999999999999" height="24" rx="5"></rect><text fill="currentColor" x="120.47999999999999" y="139" text-anchor="start">runs, then exits</text></g></g> <text fill="currentColor" x="31.8" y="18" text-anchor="start">RunJob</text> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">03:00</text></svg>

An execution runs its tasks to completion and exits. Between executions nothing runs and nothing is billed.

## Worker pools: always running, no URL

A [worker pool](worker-pools.md) keeps containers running whether or not anyone calls them, and has no URL at all. You pay for them while they run.

It fits workers that pull from Pub/Sub around the clock, or hold a connection open.

<svg viewBox="0 0 670 146" style="max-width: 770px" role="img" aria-label="A fixed number of instances run whether or not messages arrive, so there's no cold start, and you pay for every hour they run."><g><text fill="currentColor" x="4" y="34">messages</text> <g><rect stroke="currentColor" fill="none" x="154" y="18" width="33.60000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="232.4" y="18" width="33.599999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="523.6" y="18" width="33.60000000000002" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="52" x2="670" y2="52"></line><text fill="currentColor" x="4" y="78">instance 1</text> <g><rect stroke="currentColor" fill="none" x="98" y="62" width="560" height="24" rx="5"></rect><text fill="currentColor" x="378" y="78" text-anchor="middle">running · pulling from Pub/Sub</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="96" x2="670" y2="96"></line><text fill="currentColor" x="4" y="122">instance 2</text> <g><rect stroke="currentColor" fill="none" x="98" y="106" width="560" height="24" rx="5"></rect><text fill="currentColor" x="378" y="122" text-anchor="middle">running</text></g></g></svg> <svg viewBox="0 0 330 176" style="max-width: 379px" role="img" aria-label="A fixed number of instances run whether or not messages arrive, so there's no cold start, and you pay for every hour they run."><g><text fill="currentColor" x="0" y="22">messages</text> <g><rect stroke="currentColor" fill="none" x="31.8" y="32" width="19.080000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="76.32" y="32" width="19.08" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="241.68" y="32" width="19.079999999999984" height="24" rx="5"></rect></g></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="62" x2="330" y2="62"></line><text fill="currentColor" x="0" y="76">instance 1</text> <g><rect stroke="currentColor" fill="none" x="0" y="86" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="102" text-anchor="middle">running · pulling from Pub/Sub</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="116" x2="330" y2="116"></line><text fill="currentColor" x="0" y="130">instance 2</text> <g><rect stroke="currentColor" fill="none" x="0" y="140" width="318" height="24" rx="5"></rect><text fill="currentColor" x="159" y="156" text-anchor="middle">running</text></g></g></svg>

A fixed number of instances run whether or not messages arrive, so there's no cold start, and you pay for every hour they run.

## Cloud Functions: built by Google

A [Cloud Function](cloud-functions.md) is a Cloud Run service whose image Google builds from your bundled code, instead of your local Docker. Deploys take a few minutes longer, and need no Docker on the machine running them.

It fits deploying from somewhere Docker isn’t available.

Both end as a Cloud Run service. A Function skips Docker on your machine, and its deploy waits on Google's build instead.

## GKE: Kubernetes

[GKE](gke.md) runs a Kubernetes cluster, in Autopilot mode with Google managing the nodes. You deploy Deployments and Jobs from the same Stack, and bindings grant roles to each pod’s identity through Workload Identity.

A cluster takes about ten minutes to create and costs money while it exists. Choose it when you want Kubernetes itself: Helm charts, long-lived pods, or workloads you already run on it.

GKE clusterAutopilot

node 1sized by Autopilot

apiPod

workerPod

node 2sized by Autopilot

apiPod

You declare Pods through Deployments and Jobs. Kubernetes places them on nodes, and Autopilot sizes and patches the nodes for them.

## Mix runtimes in one app

An app rarely needs just one. A common shape is a service for the API, a topic it publishes to, a worker pool that consumes it, and a job that runs nightly:

<svg viewBox="0 0 628 232" style="max-width: 722px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-60cju3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 164 56 L 238 56" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-60cju3)"></path></g><g><path d="M 384 56 L 458 56" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-60cju3)"></path></g><g><path d="M 94 88 L 94 138" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-60cju3)"></path><text fill="currentColor" x="102" y="109" text-anchor="start">RunJob</text></g> <g transform="translate(24, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloud Run service</text></g> <g transform="translate(244, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Events</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Pub/Sub topic</text></g> <g transform="translate(464, 24)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">worker pool</text></g> <g transform="translate(24, 144)"><rect stroke="currentColor" fill="none" width="140" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="70" y="28" text-anchor="middle" dominant-baseline="middle">Reindex</text> <text fill="currentColor" x="70" y="46" text-anchor="middle" dominant-baseline="middle">Cloud Run job</text></g></svg> <svg viewBox="0 0 338 324" style="max-width: 389px" role="img" aria-label="Dependency graph"><defs><marker id="alc-dag-arrow-euoej8" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><path d="M 74 74 L 74 124" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-euoej8)"></path></g><g><path d="M 74 194 L 74 244" fill="none" stroke="currentColor" stroke-width="1.5" stroke-dasharray="4 4" marker-end="url(#alc-dag-arrow-euoej8)"></path></g><g><path d="M 138 42 L 194 42" fill="none" stroke="currentColor" stroke-width="1.5" marker-end="url(#alc-dag-arrow-euoej8)"></path><text fill="currentColor" x="166" y="36" text-anchor="middle">RunJob</text></g> <g transform="translate(10, 10)"><rect stroke="currentColor" fill="none" width="128" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="64" y="28" text-anchor="middle" dominant-baseline="middle">Api</text> <text fill="currentColor" x="64" y="46" text-anchor="middle" dominant-baseline="middle">Cloud Run service</text></g> <g transform="translate(10, 130)"><rect stroke="currentColor" fill="none" width="128" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="64" y="28" text-anchor="middle" dominant-baseline="middle">Events</text> <text fill="currentColor" x="64" y="46" text-anchor="middle" dominant-baseline="middle">Pub/Sub topic</text></g> <g transform="translate(10, 250)"><rect stroke="currentColor" fill="none" width="128" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="64" y="28" text-anchor="middle" dominant-baseline="middle">Worker</text> <text fill="currentColor" x="64" y="46" text-anchor="middle" dominant-baseline="middle">worker pool</text></g> <g transform="translate(200, 10)"><rect stroke="currentColor" fill="none" width="128" height="64" rx="8" ry="8"></rect><text fill="currentColor" x="64" y="28" text-anchor="middle" dominant-baseline="middle">Reindex</text> <text fill="currentColor" x="64" y="46" text-anchor="middle" dominant-baseline="middle">Cloud Run job</text></g></svg>

Each runtime runs as its own service account, granted only the roles its own bindings ask for.
