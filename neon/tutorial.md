---
url: https://alchemy.run/neon/tutorial
title: "Build an upload journal"
description: "Sign in, upload private files, process real storage events, and view Postgres-backed status with Neon and Alchemy."
access_date: 2026-09-24T22:45:48.980Z
current_date: 2026-09-24T22:45:48.980Z
---

Build a small upload application using Neon Postgres, private Object Storage, Functions, managed Better Auth, and a Vite frontend. A real storage event checks the uploaded metadata and records the result in Postgres. The browser never marks an upload ready itself.

The complete source is in [`examples/neon`](https://github.com/alchemy-run/alchemy/tree/main/examples/neon). Use that directory as the runnable project; the following pages explain its changes one concept at a time. Both the default Effect Function and the native Fetch alternative implement the same browser API.

## Prerequisites

Complete [Setup](setup.md) before deploying. Use Node.js 24, Bun, pnpm, and a Neon account with Object Storage and Functions available. Configure the Alchemy `testing` profile for Neon or supply `NEON_API_KEY` to the deployment process only. This tutorial selects `aws-us-east-2`, where the combined backend is available. Resources can incur normal Neon usage charges; no AI gateway, model call, credits, or plan purchase is required.

Install the repository’s workspace dependencies before running the example. Keep the local `.alchemy` state directory: it records which resources a later destroy command owns.

## Open the example

```sh
cd examples/neon
```

All subsequent commands run from this directory, so the migration and Function entrypoint paths resolve correctly.

## Deploy the Effect application

```sh
pnpm deploy --profile testing --stage upload-journal
```

The stack returns the website URL, API URL, Auth URL, project ID, branch ID, and bucket name. Open the website URL to create an account; it contains no database password or deployment API key.

## Use the native alternative instead

```sh
pnpm deploy:native --profile testing --stage upload-journal
```

This selects `alchemy.native.ts`, which deploys a separate stack with an ordinary asynchronous Fetch handler and an explicit `FunctionTrigger`. Choose one alternative for normal use; deploying both intentionally creates two independent backends.

## Run the frontend against a deployed backend

```sh
VITE_API_URL='<apiUrl>' VITE_NEON_AUTH_URL='<authUrl>' pnpm dev:web
```

Vite opens on `http://127.0.0.1:43187`. Managed Auth permits localhost for this tutorial, and the API defaults to wildcard CORS for bearer-token requests without cookies. Every data request still requires a verified JWT. For an origin-restricted deployment, set `UPLOAD_APP_ORIGIN` to the known frontend origin before deployment. Starting Vite does not create a fake backend or bypass authentication.

## Follow the implementation

1. [Create the database and storage](tutorial/backend.md).
2. [Protect and process uploads](tutorial/functions.md).
3. [Connect the browser](tutorial/frontend.md).
4. [Fork a preview and clean up](tutorial/previews.md).

## Understand the tutorial’s limits

Processing verifies object size and content type; it does not scan for malware. The 10 MiB limit is an application check before signing and after delivery, not an S3-enforced upload quota. A signed URL is a temporary bearer capability and can be reused until expiry. Add rate limits, upload budgets, stronger content validation, and an appropriate download domain before exposing a production service.

The example deliberately disables email verification and enables localhost. Follow [Production Auth](guides/production-auth.md) to prepare SMTP, OAuth callbacks, trusted origins, email verification, and localhost restrictions before launch. Signout revokes the managed session, but already issued JWTs can remain valid until their short expiry.
