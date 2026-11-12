---
url: https://alchemy.run/modal/networking/network
title: "Network & tunnels"
description: "Limit what a Modal Sandbox can reach, reach its ports through tunnels, and give its traffic static egress IPs."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Modal Sandbox reaches the internet unless you limit it. Modal enforces the limit outside the container, so code inside can’t lift it. Traffic into a sandbox goes through **tunnels**: each port you open gets a URL of its own.

<svg viewBox="0 0 431.4 268" style="max-width: 496px" role="img" aria-label="With allowedDomains: ['github.com'], the sandbox reaches GitHub and nothing else. Modal stops the rest outside the container."><defs><marker id="alc-bd-arrow-lyw6al" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="168" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="247.39999999999998" y="156" width="168" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="78" x2="261.4" y2="78" marker-end="url(#alc-bd-arrow-lyw6al)"></line><text fill="currentColor" x="215.7" y="70" text-anchor="middle">git clone</text></g> <g><path stroke="currentColor" fill="none" d="M 100 102 V 204 H 243.39999999999998"></path><path stroke="currentColor" fill="none" d="M 240.39999999999998 197 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="110" y="157">pip install</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">session-sam <tspan fill="currentColor" dx="8">Sandbox</tspan></text> <text fill="currentColor" x="403.4" y="175" text-anchor="end">everything else</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="75" text-anchor="middle">agent</text> <text fill="currentColor" x="100" y="91" text-anchor="middle">your code</text></g> <g><rect stroke="currentColor" fill="none" x="265.4" y="54" width="132" height="48" rx="8"></rect><text fill="currentColor" x="331.4" y="75" text-anchor="middle">github.com</text> <text fill="currentColor" x="331.4" y="91" text-anchor="middle">allowed</text></g> <g><rect stroke="currentColor" fill="none" x="265.4" y="186" width="132" height="48" rx="8"></rect><text fill="currentColor" x="331.4" y="207" text-anchor="middle">pypi.org</text> <text fill="currentColor" x="331.4" y="223" text-anchor="middle">not listed</text></g></svg>

With allowedDomains: \['github.com'\], the sandbox reaches GitHub and nothing else. Modal stops the rest outside the container.

## Block the network

```typescript
const sandbox = yield* sandboxes.create({ blockNetwork: true });
```

Nothing leaves the sandbox. Commands, files and tunnels still work, because they go through Modal.

## Allow only some destinations

```typescript
const sandbox = yield* sandboxes.create({
  allowedDomains: ["github.com", "registry.npmjs.org"],
  cidrAllowlist: ["10.0.0.0/8"],
});
```

The sandbox reaches these domains and ranges, and nothing else.

## Change it while the sandbox runs

Let an agent install its dependencies, then close the network before it runs the code it wrote:

```typescript
yield* sandbox.exec(["pnpm", "install"], { cwd: "/work" });
yield* sandbox.setNetworkAccess({ blockNetwork: true });
yield* sandbox.exec(["node", "agent-output.mjs"], { cwd: "/work" });
```

The new policy replaces the old one, and Modal closes the connections it no longer allows. Pass an allow-list to narrow the network instead, or `{}` to open it again.

## Open a port through a tunnel

List the ports when you create the sandbox. Each gets a tunnel, and `getTcpPort` returns its public URL and a client for it:

```typescript
const sandbox = yield* sandboxes.create({ encryptedPorts: [3000] });
const port = yield* sandbox.getTcpPort(3000);
const response = yield* port.fetch(HttpClientRequest.get("/health"));
```

`port.url` is the tunnel’s URL. A Worker in front of the sandbox can forward the request it received:

```typescript
return yield* port.fetch(yield* HttpServerRequest.HttpServerRequest);
```

`encryptedPorts` terminate TLS at Modal. `unencryptedPorts` pass raw TCP, for a database or SSH; `port.connect()` opens a socket to either.

A tunnel URL is public. To hand one port to a browser that shouldn’t hold any Modal credentials, create a connect token for it:

```typescript
const { url, token } = yield* sandbox.connectToken({ port: 8080, userMetadata: user });
```

The token opens that port and nothing else.

## Leave through static egress IPs

Some APIs only accept calls from known addresses. A `Modal.Proxy` gives sandboxes fixed outbound IPs:

```typescript
import * as Modal from "@alchemy.run/modal";

export const Egress = Modal.Proxy("Egress", { ips: 1 });
```

Read its id while the Worker is constructed, then pass it to `create`:

```typescript
// construction phase
const egressId = yield* (yield* Egress).proxyId;
// in a handler
const sandbox = yield* sandboxes.create({ proxy: { proxyId: yield* egressId } });
```

The Proxy’s `ips` attribute lists the addresses to allow on the other side. Proxies need Modal’s Team plan.

A route’s `redirect` sends traffic to a `Modal.Proxy` or to a sidecar container, never to a URL of yours, so a gateway such as a Cloudflare Worker can’t sit on a sandbox’s egress path. To call one, have the code in the sandbox use the gateway’s URL as its base URL.

To add an API key to a sandbox’s requests without letting it see the key, see [Secrets](../security/secrets.md#add-it-to-requests-never-into-the-sandbox).
