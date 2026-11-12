---
url: https://alchemy.run/boat/sandboxes/ports
title: "Ports & URLs"
description: "Serve a port of a Boat sandbox at an HTTPS URL, protected by a token or public, and call it or proxy to it from a Worker."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A server listening on a port inside a sandbox answers at `https://<subdomain>-<port>.on.boat.dev` once Boat hosts the port. A hosted port is **protected** by default: requests need its token. A **public** port answers anyone.

## A port is protected by its token

Without the token, the port’s URL refuses the request:

<svg viewBox="0 0 635.5999999999999 230" style="max-width: 731px" role="img" aria-label="The port's URL refuses requests without its token. getTcpPort gets the token from Boat and sends it as the _port_auth cookie on every request."><defs><marker id="alc-bd-arrow-1xk13h" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="339.2" y="118" width="280.3999999999999" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="205" y1="172" x2="353.2" y2="172" marker-end="url(#alc-bd-arrow-1xk13h)"></line><text fill="currentColor" x="281.1" y="164" text-anchor="middle">_port_auth cookie</text></g> <g><path stroke="currentColor" fill="none" d="M 205 40 H 507.0999999999999 V 114"></path><path stroke="currentColor" fill="none" d="M 500.0999999999999 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="517.0999999999999" y="83">403</text></g> <text fill="currentColor" x="351.2" y="137" text-anchor="start">box-node-…-3000.on.boat.dev <tspan fill="currentColor" dx="8">protected</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="189" height="48" rx="8"></rect><text fill="currentColor" x="110.5" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="110.5" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="189" height="48" rx="8"></rect><text fill="currentColor" x="110.5" y="169" text-anchor="middle">Api</text> <text fill="currentColor" x="110.5" y="185" text-anchor="middle">Worker · getTcpPort(3000)</text></g><g><rect stroke="currentColor" fill="none" x="357.2" y="148" width="189" height="48" rx="8"></rect><text fill="currentColor" x="451.7" y="169" text-anchor="middle">:3000</text> <text fill="currentColor" x="451.7" y="185" text-anchor="middle">npm run dev</text></g></svg> <svg viewBox="0 0 300.8 394" style="max-width: 346px" role="img" aria-label="The port's URL refuses requests without its token. getTcpPort gets the token from Boat and sends it as the _port_auth cookie on every request."><defs><marker id="alc-bd-arrow-572ofq" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="268" width="225" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="128.5" y1="212" x2="128.5" y2="308" marker-end="url(#alc-bd-arrow-572ofq)"></line><text fill="currentColor" x="138.5" y="266">_port_auth cookie</text></g> <g><path stroke="currentColor" fill="none" d="M 223 40 H 257 V 290 H 245"></path><path stroke="currentColor" fill="none" d="M 234 283 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="265" y="169">403</text></g> <text fill="currentColor" x="28" y="287" text-anchor="start">box-node-…-3000.on.boat.dev <tspan fill="currentColor" x="28" dy="14">protected</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="128.5" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="185" text-anchor="middle">Api</text> <text fill="currentColor" x="128.5" y="201" text-anchor="middle">Worker · getTcpPort(3000)</text></g><g><rect stroke="currentColor" fill="none" x="34" y="312" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="333" text-anchor="middle">:3000</text> <text fill="currentColor" x="128.5" y="349" text-anchor="middle">npm run dev</text></g></svg>

The port's URL refuses requests without its token. getTcpPort gets the token from Boat and sends it as the \_port\_auth cookie on every request.

## Call a port with its token

`getTcpPort` hosts the port and returns its URL and token. Its `fetch` sends a request there with the token:

```typescript
const port = yield* sandbox.getTcpPort(3000);
const response = yield* port.fetch(HttpClientRequest.get("/health"));
```

The request’s path, query, method, headers and body are kept, and bodies stream both ways.

## Proxy a Worker’s request to it

Pass the Worker’s own request to `fetch` to forward it to the sandbox, and return the response:

```typescript
fetch: Effect.gen(function* () {
  const port = yield* devBox.getTcpPort(3000);
  return yield* port.fetch(yield* HttpServerRequest);
});
```

## Open a port to anyone

```typescript
const port = yield* sandbox.getTcpPort(3000, { public: true });
```

Anyone with the URL can call it. For a program, set `public: true` on its `Boat.Sandbox`.

## Reach a server bound to localhost

Boat relays to a server bound to `0.0.0.0`. For one that listens on `127.0.0.1` only, pass `localhost: true`:

```typescript
const port = yield* sandbox.getTcpPort(5173, { localhost: true });
```

## Hand the URL to a browser

The token works on the `_token` query parameter as well, which answers with a redirect that sets the cookie, so a browser can open the URL:

```typescript
const port = yield* sandbox.getTcpPort(3000);
const link = \`${port.url}?_token=${Redacted.value(port.token!)}\`;
```

The token is the same every time for one sandbox and port. Treat it as a secret.

To hear when sandboxes start and stop, see [Lifecycle events](lifecycle-events.md).
