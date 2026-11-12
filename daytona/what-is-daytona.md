---
url: https://alchemy.run/daytona/what-is-daytona
title: "What is Daytona?"
description: "The concepts behind Daytona — organizations, API keys, regions and runners, snapshots, sandboxes and their lifecycle, the toolbox, previews, volumes, network limits and secrets — and how @alchemy.run/daytona models each of them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Everything in Daytona belongs to an organization, and you reach it only through Daytona’s API, with a key whose permissions Daytona checks.

Take one small app: a Worker gives each user a sandbox, booted from the `Base` snapshot, with that user’s volume as their home.

acmeorganization

Basesnapshot

usregion

user-samsandbox

user-anasandbox

home-samvolume

The organization holds its snapshots and volumes, and runs its sandboxes in a region. Each sandbox starts from a snapshot and can mount a volume. A Worker outside Daytona creates and drives the sandboxes through Daytona's API with its own key.

## Organizations: the boundary around everything

An **organization** holds your sandboxes, snapshots, volumes, secrets and API keys, the people who may use them, and the bill. Nothing in one organization can see another’s.

You sign in to Daytona once, and pick the organization your profile deploys to:

```sh
pnpm alchemy profile edit --add Daytona
```

See [Organizations and API keys](organizations-and-keys.md).

## API keys: permissions per kind of object

Every call to Daytona carries an **API key**. A key belongs to one organization and holds permissions per kind of object: `write:sandboxes` lets it create any sandbox in the organization, `manage:secrets` any secret.

Each binding creates a key of its own for its Worker or Lambda, with only the permissions its calls need:

```typescript
const sandboxes = yield* Daytona.CreateSandbox(Base);
```

See [Organizations and API keys](organizations-and-keys.md).

## Regions and runners: where sandboxes run

A **region** is where sandboxes run, such as `us` or `eu`. Inside it, **runners** are the machines that host sandboxes. Daytona runs shared regions for everyone, and you can add a region of your own and run its runners on your machines. A sandbox picks its region with `target`:

```typescript
export const DevBox = Daytona.Sandbox("DevBox", { snapshot: Base, target: "eu" });
```

See [Regions and runners](sandboxes/runners.md).

## Snapshots: the image and size a sandbox boots with

A **snapshot** is the image a sandbox boots from, plus its size: CPU, memory, disk and GPUs. Daytona builds it from an image, a Dockerfile, or another snapshot with commands run in it.

```typescript
export const Base = Daytona.Snapshot("Base", { image: "node:22-bookworm-slim", cpu: 1, memory: 1, disk: 3 });
```

See [Snapshots](sandboxes/snapshots.md).

## Sandboxes: stop, archive, start again

A **sandbox** moves through four states, and its disk survives all but the last:

<svg viewBox="-4 -55.014495755427525 776 283.0144957554275" style="max-width: 892px" role="img" aria-label="A sandbox stops after 15 idle minutes and keeps its disk. After 7 days stopped it is archived, and it can be deleted after a stop if you set autoDeleteInterval. A stopped or archived sandbox starts again with its files."><defs><marker id="alc-sm-arrow-wngpfp" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 200 63 Q 260 63 316 63" marker-end="url(#alc-sm-arrow-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 380.8333333333333 86 Q 260 231 141.72740426532457 89.07288511838951" marker-end="url(#alc-sm-arrow-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 480 63 Q 540 63 596 63" marker-end="url(#alc-sm-arrow-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 641.6666666666666 40 Q 400 -105 161.7633050361835 37.94201697828989" marker-end="url(#alc-sm-arrow-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 449.53846153846155 86 Q 540 128 626.8335025905448 168.3155547741815" marker-end="url(#alc-sm-arrow-wngpfp)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-wngpfp)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">started</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">billed while it runs</text></g> <g><rect stroke="currentColor" fill="none" x="320" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="400" y="61" text-anchor="middle">stopped</text> <text fill="currentColor" x="400" y="76" text-anchor="middle">disk kept</text></g> <g><rect stroke="currentColor" fill="none" x="600" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="680" y="61" text-anchor="middle">archived</text> <text fill="currentColor" x="680" y="76" text-anchor="middle">slower to start</text></g> <g><rect stroke="currentColor" fill="none" x="600" y="170" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="604" y="174" width="152" height="38" rx="19"></rect><text fill="currentColor" x="680" y="197" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="217.7" y="53" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="259" y="66" text-anchor="middle">idle 15 min</text></g> <g><rect stroke="currentColor" fill="none" x="239.14018439966446" y="149.26822127959738" width="43" height="17" rx="4"></rect><text fill="currentColor" x="260.64018439966446" y="162.26822127959738" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="487.8" y="53" width="102.39999999999999" height="17" rx="4"></rect><text fill="currentColor" x="539" y="66" text-anchor="middle">stopped 7 days</text></g> <g><rect stroke="currentColor" fill="none" x="379.3574929257125" y="-43.014495755427525" width="43" height="17" rx="4"></rect><text fill="currentColor" x="400.8574929257125" y="-30.014495755427525" text-anchor="middle">start</text></g> <g><rect stroke="currentColor" fill="none" x="474.6929910322516" y="117.57888869354537" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="539.0929910322516" y="130.57888869354537" text-anchor="middle">autoDeleteInterval</text></g></svg> <svg viewBox="-12.972531828189709 -4 424.9725318281897 378" style="max-width: 489px" role="img" aria-label="A sandbox stops after 15 idle minutes and keeps its disk. After 7 days stopped it is archived, and it can be deleted after a stop if you set autoDeleteInterval. A stopped or archived sandbox starts again with its files."><defs><marker id="alc-sm-arrow-n-wngpfp" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 120 86 Q 120 132 120 174" marker-end="url(#alc-sm-arrow-n-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 164.16 178 Q 252.48 132 167.70765633666792 87.84773767534787" marker-end="url(#alc-sm-arrow-n-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 120 224 Q 120 270 120 312" marker-end="url(#alc-sm-arrow-n-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 97.92 316 Q -12.47999999999999 201 95.14987268724114 88.88554928412381" marker-end="url(#alc-sm-arrow-n-wngpfp)"></path><path stroke="currentColor" fill="none" d="M 153.33333333333331 224 Q 220 270 283.37434620441667 313.7282988810475" marker-end="url(#alc-sm-arrow-n-wngpfp)"></path><g><g><circle stroke="currentColor" fill="none" cx="120" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="120" y1="20" x2="120" y2="36" marker-end="url(#alc-sm-arrow-n-wngpfp)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="61" text-anchor="middle">started</text> <text fill="currentColor" x="120" y="76" text-anchor="middle">billed while it runs</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="199" text-anchor="middle">stopped</text> <text fill="currentColor" x="120" y="214" text-anchor="middle">disk kept</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="160" height="46" rx="23"></rect><text fill="currentColor" x="120" y="337" text-anchor="middle">archived</text> <text fill="currentColor" x="120" y="352" text-anchor="middle">slower to start</text></g> <g><rect stroke="currentColor" fill="none" x="240" y="316" width="160" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="244" y="320" width="152" height="38" rx="19"></rect><text fill="currentColor" x="320" y="343" text-anchor="middle">deleted</text></g> <g><rect stroke="currentColor" fill="none" x="78.7" y="121" width="82.6" height="17" rx="4"></rect><text fill="currentColor" x="120" y="134" text-anchor="middle">idle 15 min</text></g> <g><rect stroke="currentColor" fill="none" x="209.20691408416698" y="122.46193441883696" width="43" height="17" rx="4"></rect><text fill="currentColor" x="214.20691408416698" y="135.46193441883696" text-anchor="start">start</text></g> <g><rect stroke="currentColor" fill="none" x="68.80000000000001" y="259" width="102.39999999999999" height="17" rx="4"></rect><text fill="currentColor" x="120" y="272" text-anchor="middle">stopped 7 days</text></g> <g><rect stroke="currentColor" fill="none" x="-0.9725318281897088" y="191.72138732103096" width="43" height="17" rx="4"></rect><text fill="currentColor" x="37.02746817181029" y="204.72138732103096" text-anchor="end">start</text></g> <g><rect stroke="currentColor" fill="none" x="154.7769198844375" y="259.4320747202619" width="128.8" height="17" rx="4"></rect><text fill="currentColor" x="219.1769198844375" y="272.4320747202619" text-anchor="middle">autoDeleteInterval</text></g></svg>

A sandbox stops after 15 idle minutes and keeps its disk. After 7 days stopped it is archived, and it can be deleted after a stop if you set autoDeleteInterval. A stopped or archived sandbox starts again with its files.

Create sandboxes per request or per user from a Worker, or declare a long-lived one in your Stack. See [Sandboxes](sandboxes/sandboxes.md).

## The toolbox: the API inside every sandbox

The **toolbox** is the API inside every sandbox. Alchemy hands you a sandbox as a handle with all of it:

```typescript
const result = yield* sandbox.exec("npm test", { cwd: "/home/daytona/repo" });
```

See [Toolbox](sandboxes/toolbox.md).

## Previews: a URL for each port

A **preview URL** reaches a port inside a sandbox from outside, through Daytona’s proxy. It needs the sandbox’s preview token unless the sandbox is `public`. A port’s `fetch` sends the token for you:

```typescript
const port = yield* sandbox.getTcpPort(3000);
const res = yield* port.fetch(HttpClientRequest.get("/health"));
```

See [Network and previews](networking/network.md).

## Volumes: files that outlive the sandbox

A **volume** is a persistent disk, backed by object storage, that sandboxes mount at a path. Its files outlive every sandbox that mounts it, so a user’s home can survive their sandbox.

```typescript
export const Cache = Daytona.Volume("Cache", {});
```

See [Volumes](sandboxes/volumes.md).

## Network limits: what a sandbox can reach

Daytona controls what a sandbox can reach, outside the sandbox. Your organization’s tier sets the default: below Tier 3, sandboxes reach only an allowlist of essential services, such as package registries, GitHub and AI APIs. On top of that, a sandbox can allow only some domains or ranges, or no network at all:

```typescript
export const DevBox = Daytona.Sandbox("DevBox", { snapshot: Base, domainAllowList: ["github.com"] });
```

See [Network and previews](networking/network.md).

## Secrets: values a sandbox uses but can’t read

A **secret** is a value a sandbox can use but not read. The sandbox sees a placeholder, and Daytona’s egress proxy swaps in the real value only on requests to the hosts you name:

```typescript
export const NpmToken = Daytona.Secret("NpmToken", {
  value: Config.Redacted("NPM_TOKEN"),
  hosts: ["registry.npmjs.org"],
});
```

See [Secrets](security/secrets.md).

## State: where Alchemy keeps its record

Daytona has no storage for Alchemy’s own record of what each Stack created. Keep it on disk with `Alchemy.localState()`, or share it with your team and CI through `Cloudflare.state()` or `AWS.state()`.

## How @alchemy.run/daytona models these concepts

Snapshots, volumes, secrets, registries, warm pools, regions, runners, roles and organization settings are resources you declare in a Stack. A sandbox is a resource too when it should live as long as the Stack, like a dev box, or a sandbox that runs a program you write. Sandboxes, volumes, secrets and snapshots that come and go per request or per user are created at runtime through bindings (`CreateSandbox`, `CreateVolume`, `CreateSecret`, `CreateSnapshot`), each with its own API key. Your [profile](../environments/profiles.md) signs you in to Daytona and picks the organization.

Next, build one: [set up](setup.md) your profile, then the [tutorial](tutorial/part-1.md) puts these ideas to work, one step at a time.
