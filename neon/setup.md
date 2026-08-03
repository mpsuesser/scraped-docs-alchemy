---
url: https://alchemy.run/neon/setup
title: "Setup"
description: "Connect alchemy to Neon — account, credentials, and profiles."
access_date: 2026-08-03T18:22:56.523Z
current_date: 2026-08-03T18:22:56.523Z
---

Sign up at [neon.tech](https://neon.tech) and create an API key in
the [Neon console](https://console.neon.tech). Register the provider
next to your cloud's:

```typescript
// alchemy.run.ts
import * as Neon from "alchemy/Neon";

providers: Layer.mergeAll(Cloudflare.providers(), Neon.providers()),
```

The next `alchemy login` adds a `Neon` step with two options:

- **Environment variable** — reads `NEON_API_KEY` (good for CI).
- **Stored API key** — entered interactively, saved under
  `~/.alchemy/credentials/<profile>/neon-stored.json`.

In CI (`CI=true`), the environment-variable method is selected
automatically — no prompt, just set `NEON_API_KEY`.

See [Profiles](../environments/profiles.md) for how credentials are stored
and switched.

## Next steps

- [Neon overview](../neon.md) — resources and compositions.
