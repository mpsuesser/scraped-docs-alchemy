---
url: https://alchemy.run/archil/security/network
title: "Network"
description: "Control what a sandbox can reach, and add credentials to its requests on the way out."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Every sandbox has an outbound policy. Archil enforces it outside the microVM, so code inside can’t change it, and it can add headers to requests as they leave, so a key the sandbox uses never sits inside it.

## The policy is enforced outside the microVM

<svg viewBox="0 0 833.8 268" style="max-width: 959px" role="img" aria-label="With default: deny, a sandbox reaches only the hosts it allows. The policy adds the Authorization header to requests for api.github.com, so the token never enters the sandbox."><defs><marker id="alc-bd-arrow-d77kj2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="225" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="592.8" y="156" width="225" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="128.5" y1="102" x2="128.5" y2="182" marker-end="url(#alc-bd-arrow-d77kj2)"></line><text fill="currentColor" x="138.5" y="148">GET /user</text></g> <g><path stroke="currentColor" fill="none" d="M 223 78 H 705.3 V 152"></path><path stroke="currentColor" fill="none" d="M 698.3 149 l 14 14 m 0 -14 l -14 14"></path></g><text fill="currentColor" x="28" y="43" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">egress: deny</tspan></text> <text fill="currentColor" x="805.8" y="175" text-anchor="end">everything else</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="75" text-anchor="middle">agent</text> <text fill="currentColor" x="128.5" y="91" text-anchor="middle">your code</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="207" text-anchor="middle">api.github.com</text> <text fill="currentColor" x="128.5" y="223" text-anchor="middle">allowed · + Authorization</text></g> <g><rect stroke="currentColor" fill="none" x="610.8" y="186" width="189" height="48" rx="8"></rect><text fill="currentColor" x="705.3" y="207" text-anchor="middle">example.com</text> <text fill="currentColor" x="705.3" y="223" text-anchor="middle">not listed</text></g></svg> <svg viewBox="0 0 273 432" style="max-width: 314px" role="img" aria-label="With default: deny, a sandbox reaches only the hosts it allows. The policy adds the Authorization header to requests for api.github.com, so the token never enters the sandbox."><defs><marker id="alc-bd-arrow-qzjsdf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="225" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="320" width="225" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="128.5" y1="102" x2="128.5" y2="198" marker-end="url(#alc-bd-arrow-qzjsdf)"></line><text fill="currentColor" x="138.5" y="156">GET /user</text></g> <g><path stroke="currentColor" fill="none" d="M 223 78 H 257 V 342 H 245"></path><path stroke="currentColor" fill="none" d="M 234 335 l 14 14 m 0 -14 l -14 14"></path></g><text fill="currentColor" x="28" y="43" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">egress: deny</tspan></text> <text fill="currentColor" x="229" y="339" text-anchor="end">everything else</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="75" text-anchor="middle">agent</text> <text fill="currentColor" x="128.5" y="91" text-anchor="middle">your code</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="223" text-anchor="middle">api.github.com</text> <text fill="currentColor" x="128.5" y="239" text-anchor="middle">allowed · + Authorization</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="371" text-anchor="middle">example.com</text> <text fill="currentColor" x="128.5" y="387" text-anchor="middle">not listed</text></g></svg>

With default: deny, a sandbox reaches only the hosts it allows. The policy adds the Authorization header to requests for api.github.com, so the token never enters the sandbox.

## Deny everything but a list of hosts

```typescript
export const MySandbox = Archil.Sandbox("MySandbox", {
  network: {
    egress: { default: "deny", allow: ["registry.npmjs.org", "*.github.com"] },
  },
});
```

Deny entries win over allow entries. A wildcard matches subdomains, not the apex domain.

## Add a credential on the way out

An allow entry can set headers on matching HTTPS requests. The sandbox never sees the value:

```typescript
export const MySandbox = Archil.Sandbox("MySandbox", {
  network: {
    egress: {
      default: "deny",
      allow: [
        { target: "api.github.com", headers: { Authorization: Config.Redacted("GITHUB_AUTH") } },
      ],
    },
  },
});
```

Archil has no secret store, so the value lives in the sandbox’s policy.

## Send requests through your own gateway

An allow entry can also forward matching requests to a URL of yours. Archil decrypts the request, applies the entry’s headers, and sends it to `forwardUrl` with the original path and query. It names the original destination and the calling sandbox in `archil-forwarded-host`, `archil-forwarded-scheme`, `archil-forwarded-port`, `archil-forwarded-path` and `archil-sandbox-id` headers.

<svg viewBox="0 0 848.8 136" style="max-width: 976px" role="img" aria-label="The agent calls api.example.com as usual. Archil forwards the request to the gateway and adds the token that proves it came from this policy. The gateway adds the API key and calls the real host. Neither secret enters the sandbox."><defs><marker id="alc-bd-arrow-hszejw" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="192" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="190" y1="78" x2="351.4" y2="78" marker-end="url(#alc-bd-arrow-hszejw)"></line><text fill="currentColor" x="272.7" y="70" text-anchor="middle">forwarded · + token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="511.4" y1="78" x2="672.8" y2="78" marker-end="url(#alc-bd-arrow-hszejw)"></line><text fill="currentColor" x="594.0999999999999" y="70" text-anchor="middle">calls</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">egress: deny</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="75" text-anchor="middle">agent</text> <text fill="currentColor" x="112" y="91" text-anchor="middle">your code</text></g> <g><rect stroke="currentColor" fill="none" x="355.4" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="433.4" y="75" text-anchor="middle">MyGateway</text> <text fill="currentColor" x="433.4" y="91" text-anchor="middle">Worker · + x-api-key</text></g> <g><rect stroke="currentColor" fill="none" x="676.8" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="754.8" y="75" text-anchor="middle">api.example.com</text> <text fill="currentColor" x="754.8" y="91" text-anchor="middle">receives the key</text></g></svg> <svg viewBox="0 0 263.4 414" style="max-width: 303px" role="img" aria-label="The agent calls api.example.com as usual. Archil forwards the request to the gateway and adds the token that proves it came from this policy. The gateway adds the API key and calls the real host. Neither secret enters the sandbox."><defs><marker id="alc-bd-arrow-unocan" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="192" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="112" y1="102" x2="112" y2="198" marker-end="url(#alc-bd-arrow-unocan)"></line><text fill="currentColor" x="122" y="156">forwarded · + token</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="112" y1="250" x2="112" y2="346" marker-end="url(#alc-bd-arrow-unocan)"></line><text fill="currentColor" x="122" y="304">calls</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">sandbox <tspan fill="currentColor" dx="8">egress: deny</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="75" text-anchor="middle">agent</text> <text fill="currentColor" x="112" y="91" text-anchor="middle">your code</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="223" text-anchor="middle">MyGateway</text> <text fill="currentColor" x="112" y="239" text-anchor="middle">Worker · + x-api-key</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="371" text-anchor="middle">api.example.com</text> <text fill="currentColor" x="112" y="387" text-anchor="middle">receives the key</text></g></svg>

The agent calls api.example.com as usual. Archil forwards the request to the gateway and adds the token that proves it came from this policy. The gateway adds the API key and calls the real host. Neither secret enters the sandbox.

### Forward the API’s requests to the gateway

```typescript
const gateway = yield* MyGateway;
const sandbox = yield* Archil.Sandbox("MySandbox", {
  network: {
    egress: {
      default: "deny",
      allow: [
        {
          target: "api.example.com",
          headers: { "X-Gateway-Token": Config.Redacted("GATEWAY_TOKEN") },
          forwardUrl: gateway.url.as<string>(),
        },
      ],
    },
  },
});
```

Requests for `api.example.com` now go to the gateway. Archil adds the gateway token on the way, so the gateway can tell they came through this policy.

### Give the gateway its secrets

```typescript
export default class MyGateway extends Cloudflare.Worker<MyGateway>()(
  "MyGateway",
  {
    main: import.meta.url,
    env: {
      GATEWAY_TOKEN: Config.Redacted("GATEWAY_TOKEN"),
      API_KEY: Config.Redacted("API_KEY"),
    },
  },
  Effect.succeed({ fetch: forward }),
) {}
```

A Cloudflare Worker makes a small gateway. It holds the token it expects and the API key it adds. `forward` is the handler the next steps build.

### Refuse requests without the token

```typescript
const request = yield* HttpServerRequest;
const env = yield* Cloudflare.Workers.WorkerEnvironment;
if (request.headers["x-gateway-token"] !== env.GATEWAY_TOKEN) {
  return HttpServerResponse.text("forbidden", { status: 403 });
}
```

The gateway’s URL is public, so anyone can call it. Only Archil adds the token.

### Send on only to hosts you list

```typescript
const host = request.headers["archil-forwarded-host"];
if (host === undefined || !["api.example.com"].includes(host)) {
  return HttpServerResponse.text("host not allowed", { status: 403 });
}
```

Without the list, the gateway would add your key to requests for any host.

### Add the key and call the real host

```typescript
const { pathname, search } = new URL(request.originalUrl);
const client = yield* HttpClient.HttpClient;
const upstream = yield* client.execute(
  HttpClientRequest.get(\`https://${host}${pathname}${search}\`).pipe(
    HttpClientRequest.setHeader("x-api-key", env.API_KEY),
  ),
);
return HttpServerResponse.text(yield* upstream.text, { status: upstream.status });
```

Archil appends the original path and query to `forwardUrl`, so the gateway rebuilds the URL from them. For other methods, bodies and headers, copy them from `request`, leaving out the `archil-` headers and the token.

The gateway holds the API key, so rotating it changes only the Worker. `archil-sandbox-id` tells it which sandbox called, which it can use to log, rate-limit or pick a key per sandbox. `forwardUrl` must be a public HTTPS URL on a domain name, so a `workers.dev` URL or a custom domain works, and an IP address doesn’t.

## Change the policy while it runs

```typescript
yield* sandbox.setEgress({ default: "deny", allow: ["api.openai.com"] });
```

The new policy applies at once. On a declared `Archil.Sandbox`, changing `network` updates it in place.

## Your plan sets the default

On Archil’s Free plan, sandboxes reach only a managed allowlist of package registries, Git hosts and model APIs, and adding headers needs the Developer plan or above. A sandbox created on a paid plan with no policy reaches anything.
