---
url: https://alchemy.run/github
title: "GitHub"
description: "Repositories, Actions secrets and variables, webhooks, and repository event sources as Stack resources — the glue for CI/CD."
access_date: 2026-08-03T19:00:17.443Z
current_date: 2026-08-03T19:00:17.443Z
---

The GitHub provider manages repositories, Actions secrets and variables, and webhooks as resources. It’s the glue for CI/CD: your Stack can mint a scoped cloud credential and store it as an Actions secret in the same deploy — no pasting tokens into repo settings. Workers can even subscribe to repository events directly.

New here? [Set up credentials](github/setup.md) first.

## Resources

`Repository` creates and converges a repo’s settings — see [Repositories](github/repository.md). Destroying the stack retains the repository by default, protecting its history:

```typescript
const repo = yield* GitHub.Repository("api", {
  owner: "my-org",
  name: "api",
  description: "API service",
  autoInit: true,
});
```

`Secrets` and `Variables` bulk-seed GitHub Actions configuration — values can be outputs of other resources. See [Actions secrets & variables](github/actions-config.md):

```typescript
yield* GitHub.Secrets({
  owner: "my-org",
  repository: "api",
  secrets: {
    AXIOM_INGEST_TOKEN: tokenValue,
  },
});

yield* GitHub.Variables({
  owner: "my-org",
  repository: "api",
  variables: {
    AWS_ROLE_ARN: role.roleArn,
    AWS_REGION: region,
  },
});
```

`Environment` manages a deployment environment and its protection rules; secrets and variables scope to it via their `environment` prop:

```typescript
const production = yield* GitHub.Environment("production", {
  owner: "my-org",
  repository: "api",
  name: "production",
  reviewers: { teams: ["platform"] },
});
```

`Webhook` manages a repository webhook pointed at any URL, and `consumeRepositoryEvents` subscribes a host (e.g. a Cloudflare Worker) to repository webhooks with typed payloads — see [Webhooks & events](github/events.md) and [React to GitHub events from a Worker](cloudflare/messaging/github-events.md):

```typescript
yield* GitHub.consumeRepositoryEvents(
  {
    owner: "my-org",
    repository: "my-repo",
    events: ["push", "pull_request"],
    secret,
  },
  (event) => Effect.log(\`received ${event.name} (${event.id})\`),
);
```

## PR comments

`Comment` posts a sticky comment on an issue or pull request: created on the first deploy, updated in place on every subsequent deploy because the logical ID stays the same, and never deleted on destroy by default (set `allowDelete: true` to opt in). Use `Output.interpolate` to embed attributes that aren’t resolved yet — like a preview URL:

```typescript
if (process.env.PULL_REQUEST) {
  yield* GitHub.Comment("preview-comment", {
    owner: "my-org",
    repository: "my-repo",
    issueNumber: Number(process.env.PULL_REQUEST),
    body: Output.interpolate\`
      ## Preview Deployed

      **URL:** ${website.url}
    \`,
  });
}
```

The body is dedented automatically, so indented template literals render as clean Markdown. [CI](environments/ci.md) walks through the full workflow: a `pr-{n}` stage per pull request, with this comment tracking its preview URL on each push.

## Compose with your cloud

The canonical pattern: a one-shot stack mints a scoped cloud credential and stores it as an Actions secret, so CI deploys with a credential that was itself provisioned as code. Walk through it in [Part 5: CI/CD (Cloudflare)](cloudflare/tutorial/part-5.md) or [Part 5: CI/CD (AWS)](aws/tutorial/part-5.md), and see [CI](environments/ci.md) for the general GitHub Actions setup.

## Reference

[Repository](https://alchemy.run/providers/github/repository) · [Environment](https://alchemy.run/providers/github/environment) · [Secret](https://alchemy.run/providers/github/secret) · [Secrets](https://alchemy.run/providers/github/secrets) · [Variable](https://alchemy.run/providers/github/variable) · [Variables](https://alchemy.run/providers/github/variables) · [Webhook](https://alchemy.run/providers/github/webhook) · [Comment](https://alchemy.run/providers/github/comment)
