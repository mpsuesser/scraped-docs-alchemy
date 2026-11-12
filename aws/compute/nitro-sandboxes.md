---
url: https://alchemy.run/aws/compute/nitro-sandboxes
title: "Nitro Sandboxes"
description: "How Nitro Sandboxes run each ECS task, EKS pod or self-managed workload in its own lightweight VM on your EC2 instances, the difference between isolated and confidential modes, and how to turn them on for ECS, EKS and EC2."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Containers on the same machine share one Linux kernel. A container escape or a kernel bug can let one workload reach another. A **Nitro Sandbox** closes that gap: it is a lightweight virtual machine, with its own kernel, that the Nitro hypervisor isolates from the instance it runs on and from every other sandbox there.

Sandboxes run on your own EC2 instances, so many of them share one machine. Use them for code you don’t trust: a tenant’s container, an agent running code an LLM wrote, a CI job from a pull request.

## One VM per task or pod

Turn the feature on for an instance, and each task or pod scheduled onto it starts in its own sandbox. The instance gives the sandbox its vCPUs and memory, and Nitro keeps them separate.

m6i.xlargeEC2 instance · Nitro Enclaves enabled

user-42Nitro Sandbox · own kernel

appcontainer

user-43Nitro Sandbox · own kernel

appcontainer

ci-runNitro Sandbox · own kernel

runnercontainer

Each task or pod gets its own sandbox on the shared instance. Nothing inside one sandbox can reach another sandbox or the instance.

The instance must be a type that supports Nitro Enclaves, which is most current general purpose, compute and memory families above their smallest size (`m6i.xlarge` yes, `m6i.large` no). The vCPUs of all sandboxes on an instance can add up to five times the instance’s own. Memory is not shared: each sandbox’s memory is set aside for it until it stops. See [supported instance types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-ec2-nitro-sandboxes.html#nitro-sandboxes-considerations).

## Isolated or confidential

A sandbox runs in one of two modes, depending on who you trust.

**Isolated** trusts the instance and distrusts the workload. It protects the instance and its neighbours from what runs inside. The instance can still exec into the container, read its logs and see its processes.

**Confidential** distrusts the instance too. Nitro moves the sandbox’s memory out of the instance’s reach, and the sandbox can prove which code it runs through **attestation**. You write a KMS key policy that releases a key only to a sandbox whose measurements match, so secrets reach exactly the code you approved. Confidential workloads need a policy that pins their images, arguments and mounts, and exec is denied.

Start with isolated. Move to confidential when the people or software operating the instance must not see the data.

## On ECS

ECS runs sandboxes on **Managed Instances**: a capacity provider whose instances ECS launches and patches. Enable sandboxes on its instances, then set the isolation mode on the task definition:

```typescript
const capacity = yield* AWS.ECS.CapacityProvider("Capacity", {
  cluster: cluster.clusterName,
  managedInstancesProvider: {
    instanceLaunchTemplate: {
      networkConfiguration: { subnets: network.privateSubnetIds },
      nitroEnclavesEnabled: true,
    },
  },
});

const task = yield* AWS.ECS.TaskDefinition("MyTask", {
  requiresCompatibilities: ["MANAGED_INSTANCES"],
  trustedExecutionConfiguration: { isolationMode: "ISOLATED_SANDBOX" },
  containerDefinitions: [{ name: "app", image: "ghcr.io/acme/app:1" }],
});
```

Running it as a Service is in [ECS: Isolate each task in its own VM](ecs.md#isolate-each-task-in-its-own-vm).

ECS isolates the task and stops there. `AWS.ECS.CreateSandbox` is Alchemy’s construct on top: your app starts sandboxes while it runs, one per user or session, and runs commands, uses files and calls ports in each through a handle. See [ECS Sandboxes](ecs-sandboxes.md).

## On EKS

On an EKS Auto Mode cluster, set `runtimeClassName` on a Deployment or a Job. Auto Mode provisions a node that supports sandboxes and starts each pod in its own:

```typescript
yield* Kubernetes.Deployment("MyApp", {
  cluster,
  image: "ghcr.io/acme/app:1",
  port: 8080,
  runtimeClassName: "isolated-sandbox", // or "confidential-sandbox"
});
```

See [EKS: Run each pod in its own VM](eks.md#run-each-pod-in-its-own-vm).

## On EC2

On a plain instance, enable Nitro Enclaves at launch. You then run your own virtual machine monitor on it, usually Firecracker, and start sandboxes yourself:

```typescript
yield* AWS.EC2.Instance("SandboxHost", {
  imageId: AWS.EC2.amazonLinux2023(),
  instanceType: "m6i.xlarge",
  enclaveOptions: { enabled: true },
});
```

See [EC2: Host Nitro Sandboxes](ec2.md#host-nitro-sandboxes).

## Nitro Sandboxes or Lambda MicroVMs

Both give each workload its own VM. They differ in who runs the machines.

Nitro Sandboxes run on instances in your account, inside ECS or EKS. You keep the cluster and the network, and pay for each instance while it runs, however many sandboxes share it. They suit a fleet you already run on containers, and work that must stay on machines in your own account.

[Lambda MicroVMs](microvms.md) run on AWS’s machines. You build an image, launch one MicroVM per user or session from a Function, and drive it over HTTP. There are no instances or cluster to run. They suit sandboxes started on demand when you’d rather not run the machines.
