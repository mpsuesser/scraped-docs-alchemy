---
url: https://alchemy.run/cloudflare/security/turnstile
title: "Turnstile CAPTCHA"
description: "Manage Cloudflare Turnstile widgets as resources — deploy a widget, embed its sitekey in HTML, and verify tokens server-side with the redacted secret."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Turnstile is Cloudflare’s CAPTCHA alternative. A Turnstile widget is defined by the hostnames it runs on and its interaction mode; Cloudflare assigns two keys in return — a public `sitekey` you embed in HTML and a `secret` your server uses to verify the tokens the widget produces.

<svg viewBox="0 0 570.2 230" style="max-width: 656px" role="img" aria-label="The sitekey in the page only renders the widget. The Worker holds the secret and checks each token with Cloudflare before it trusts the form."><defs><marker id="alc-bd-arrow-vn5ncn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="249.2" y="118" width="305.00000000000006" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="148" y1="172" x2="263.2" y2="172" marker-end="url(#alc-bd-arrow-vn5ncn)"></line><text fill="currentColor" x="207.6" y="164" text-anchor="middle">form + token</text></g> <g><path stroke="currentColor" fill="none" d="M 148 40 H 470.20000000000005 V 114"></path><path stroke="currentColor" fill="none" d="M 463.20000000000005 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="480.20000000000005" y="83">rejected</text></g> <text fill="currentColor" x="261.2" y="137" text-anchor="start">Signup <tspan fill="currentColor" dx="8">Worker · siteverify with the secret</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="82" y="37" text-anchor="middle">bot</text> <text fill="currentColor" x="82" y="53" text-anchor="middle">no valid token</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="132" height="48" rx="8"></rect><text fill="currentColor" x="82" y="169" text-anchor="middle">browser</text> <text fill="currentColor" x="82" y="185" text-anchor="middle">widget token</text></g> <g><rect stroke="currentColor" fill="none" x="267.2" y="148" width="132" height="48" rx="8"></rect><text fill="currentColor" x="333.2" y="169" text-anchor="middle">POST /signup</text> <text fill="currentColor" x="333.2" y="185" text-anchor="middle">handled</text></g></svg> <svg viewBox="0 0 363.8 394" style="max-width: 418px" role="img" aria-label="The sitekey in the page only renders the widget. The Worker holds the secret and checks each token with Cloudflare before it trusts the form."><defs><marker id="alc-bd-arrow-v5oaun" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="268" width="255" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="100" y1="212" x2="100" y2="308" marker-end="url(#alc-bd-arrow-v5oaun)"></line><text fill="currentColor" x="110" y="266">form + token</text></g> <g><path stroke="currentColor" fill="none" d="M 166 40 H 287 V 290 H 275"></path><path stroke="currentColor" fill="none" d="M 264 283 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="295" y="169">rejected</text></g> <text fill="currentColor" x="28" y="287" text-anchor="start">Signup <tspan fill="currentColor" x="28" dy="14">Worker · siteverify with the secret</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="37" text-anchor="middle">bot</text> <text fill="currentColor" x="100" y="53" text-anchor="middle">no valid token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="185" text-anchor="middle">browser</text> <text fill="currentColor" x="100" y="201" text-anchor="middle">widget token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="333" text-anchor="middle">POST /signup</text> <text fill="currentColor" x="100" y="349" text-anchor="middle">handled</text></g></svg>

The sitekey in the page only renders the widget. The Worker holds the secret and checks each token with Cloudflare before it trusts the form.

## Create a widget

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "SignupApp",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state(),
  },
  Effect.gen(function* () {
    const widget = yield* Cloudflare.Turnstile.Widget("SignupForm", {
      domains: ["example.com"],
      mode: "managed",
    });

    return {
      sitekey: widget.sitekey,
    };
  }),
);
```

`domains` lists the hostnames the widget may run on — subdomains are covered automatically. `mode` is `managed` (Cloudflare decides whether an interaction is required), `non-interactive` (never asks), or `invisible` (no visible widget). If you omit `name`, a unique one is generated from the app, stage, and logical ID.

## The two keys

```typescript
const sitekey = widget.sitekey; // public — render it in your HTML
const secret = widget.secret; // Redacted<string> — server-side only
```

The `sitekey` is public and safe to expose as a stack output. The `secret` is a `Redacted<string>`: it never prints in logs or JSON-serialized output (it renders as `"<redacted>"`), so unwrap it with `Redacted.value` at the point where your server actually needs the raw string.

## Embed the sitekey

```html
<script
  src="https://challenges.cloudflare.com/turnstile/v0/api.js"
  async
  defer
></script>
<div class="cf-turnstile" data-sitekey="<sitekey from the deploy output>"></div>
```

The widget runs the challenge in the browser and puts the resulting token in a hidden `cf-turnstile-response` form field (or hands it to your callback). The token proves nothing by itself — your server must verify it.

## Verify tokens server-side

There is no runtime binding for Turnstile — verification is a plain HTTPS call from any backend. POST the token together with `Redacted.value(secret)` to Cloudflare’s `https://challenges.cloudflare.com/turnstile/v0/siteverify` endpoint and check `success` in the response; see Cloudflare’s [server-side validation docs](https://developers.cloudflare.com/turnstile/get-started/server-side-validation/) for the full request/response contract. A Worker is a natural place to do this — pass the secret in via the Worker’s `env`.

## Updates and replacement

```typescript
const widget = yield* Cloudflare.Turnstile.Widget("SignupForm", {
  domains: ["example.com", "app.example.com"],
  mode: "invisible",
});
```

Name, domains, mode, and the clearance settings all mutate in place — the widget keeps its `sitekey` and `secret`, so deployed pages keep working. Only `region` (default `"world"`) forces a replacement, which mints a new sitekey and secret. If the widget is deleted out-of-band, the next deploy observes it missing and recreates it rather than failing.

## Where next

- [Workers](../compute/workers.md) — host your siteverify call next to the form it protects.
- [Secrets & env](secrets-env.md) — how the widget secret gets into the Worker.
- [Widget API reference](https://alchemy.run/providers/cloudflare/turnstile#widget)
