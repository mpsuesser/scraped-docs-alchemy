---
url: https://alchemy.run/neon/setup
title: "Setup"
description: "Connect alchemy to Neon — account, credentials, and profiles."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

Sign up at [neon.tech](https://neon.tech) and create an API key in
the [Neon console](https://console.neon.tech). Register the provider
next to your cloud's:

```typescript
// alchemy.run.ts
import * as Neon from "alchemy/Neon";

providers: Layer.mergeAll(Cloudflare.providers(), Neon.providers()),
```

Run `alchemy profile edit --add Neon`. The API key is entered interactively
and saved under `~/.alchemy/credentials/<profile>/neon-stored.json`.

In CI (`CI=true`), Alchemy skips profiles. Set `NEON_API_KEY` and Alchemy
reads it directly without persisting anything.

See [Profiles](../environments/profiles.md) for how credentials are stored
and switched.

## Next steps

- [Neon overview](../neon.md) — resources and compositions.
