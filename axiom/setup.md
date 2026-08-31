---
url: https://alchemy.run/axiom/setup
title: "Setup"
description: "Connect alchemy to Axiom — account, credentials, and profiles."
access_date: 2026-08-31T21:01:48.980Z
current_date: 2026-08-31T21:01:48.980Z
---

Sign up at [axiom.co](https://axiom.co) and create an API token in
the [Axiom app](https://app.axiom.co). Register the provider next to
your cloud's:

```typescript
// alchemy.run.ts
import * as Axiom from "alchemy/Axiom";

providers: Layer.mergeAll(Cloudflare.providers(), Axiom.providers()),
```

Run `alchemy profile edit --add Axiom` and choose an API token or personal
access token. It is entered interactively and saved under
`~/.alchemy/credentials/<profile>/`.

In CI (`CI=true`), profiles are bypassed. Set `AXIOM_TOKEN` or
`AXIOM_API_KEY`; `AXIOM_ORG_ID` and `AXIOM_URL` are optional.

See [Profiles](../environments/profiles.md) for how credentials are stored
and switched.

## Next steps

- [Axiom overview](../axiom.md) — resources and compositions.
