---
url: https://alchemy.run/hetzner/projects-and-tokens
title: "Projects, tokens and SSH keys"
description: "How a Hetzner Cloud project isolates your Servers, how its API token decides where Alchemy deploys, and how SSH keys reach your Servers."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything in Hetzner Cloud lives inside a **project**: Servers, volumes, networks, firewalls, IP addresses and the tokens that manage them. Nothing in one project can see another.

my-app-prodprojecttoken: my-app-prod

web-1Server · fsn1

dataVolume

webFirewall

my-app-devprojecttoken: my-app-dev

web-samServer · fsn1

web-pr-42Server · fsn1

Each project holds its own servers, disks and firewalls, and each token reaches one project. Many stages share the development project.

## A token reaches one project

An **API token** is created inside a project, under **Security → API tokens**, and reaches only that project. So the token is how you choose where Alchemy deploys. Give it **Read & Write** access, so Alchemy can create resources, and store it in a [profile](../environments/profiles.md):

```sh
alchemy profile edit --add Hetzner
```

See [Setup](setup.md) for the details.

## One project per environment

Because a token reaches one project, separate projects give you hard isolation. Use one for production and another for everything else, each with its own token and profile:

```sh
alchemy profile create prod
alchemy profile edit --profile prod --add Hetzner
alchemy deploy --stage prod --profile prod
```

Many [stages](../environments/stages.md) can share the development project, each with its own Servers.

## SSH keys

A Server is a whole machine, and you sign in to it over SSH. An **SSH key** registers your public key with the project, so Servers created with it accept your key, and never a password:

```typescript
const key = yield* Hetzner.SshKey("deploy", {
  publicKey: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI… you@laptop",
});
```

See [Servers](compute/servers.md) for attaching keys to a Server.

## In CI

With `CI=true`, Alchemy skips the profile and reads `HCLOUD_TOKEN` from the environment. See [CI](../environments/ci.md).
