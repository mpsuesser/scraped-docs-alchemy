---
url: https://alchemy.run/github
title: "GitHub"
description: "Repositories, Actions secrets, variables and environments, webhooks, repository event sources and pull request comments as Stack resources — the glue for CI/CD."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

GitHub hosts git repositories and the work around them: reviewing changes in pull requests, tracking issues, and running CI/CD with Actions.

Actions builds, tests and deploys on every push, using credentials kept in its secrets and variables. Environments guard production behind reviews. Webhooks tell other systems when something happens in a repository.

**New to GitHub?** Start with [What is GitHub?](github/what-is-github.md).

## A quick example

A repository, with its settings kept in code:

```typescript
const repo = yield* GitHub.Repository("api", { owner: "my-org", name: "api", autoInit: true });
```

Actions variables set from another resource’s outputs:

```typescript
yield* GitHub.Variables({
  owner: "my-org",
  repository: "api",
  variables: { AWS_ROLE_ARN: role.roleArn, AWS_REGION: region },
});
```

And one comment per pull request, updated on every deploy with the preview’s URL:

```typescript
yield* GitHub.Comment("preview-comment", {
  owner: "my-org",
  repository: "api",
  issueNumber: yield* Config.Number("PULL_REQUEST"),
  body: Output.interpolate\`Preview: ${website.url}\`,
});
```

## Where to start

**Want to build?** Connect your token in [Setup](github/setup.md), then [CI](environments/ci.md) walks through a GitHub Actions workflow with a stage per pull request.

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Accounts & access

- [Accounts and tokens](github/accounts-and-tokens.md)
- [Actions secrets & variables](github/actions-config.md)
