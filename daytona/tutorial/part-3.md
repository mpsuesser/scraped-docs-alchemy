---
url: https://alchemy.run/daytona/tutorial/part-3
title: "Part 3: A Copy per User"
description: "Give each user a Daytona sandbox started from the warm snapshot, which stops when idle and starts again with their changes."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

## Start a user’s sandbox from Warm

```typescript
const userSandbox = Effect.fn("userSandbox")(function* (user: string) {
  const existing = yield* sandboxes.get(\`dev-${user}\`);
  if (existing === undefined) {
    return yield* sandboxes.create({ name: \`dev-${user}\`, labels: { user }, autoStopInterval: 15 });
  }
  yield* existing.startSandbox();
  return existing;
});
```

A user’s first request starts their sandbox from `Warm`, with the repository already cloned. It’s named after the user, so `sandboxes.get` finds it again.

## Their changes stay in their copy

```typescript
const match = /^\/users\/([a-z0-9-]+)\/files\/(.+)$/.exec(url.pathname);
if (match !== null) {
  const sandbox = yield* userSandbox(match[1]!);
  const path = \`/home/daytona/repo/${match[2]}\`;
  if (request.method === "PUT") yield* sandbox.files.write(path, yield* request.text);
  return yield* HttpServerResponse.json({ content: yield* sandbox.files.readText(path) });
}
```

Each user edits their own copy of the repository. `Warm` never changes, so the next sandbox started from it doesn’t see their edits.

## Stop when idle, start where they left off

`autoStopInterval: 15` stops the sandbox after 15 idle minutes, and Daytona keeps its disk. `startSandbox` on their next request starts it again with their files, and does nothing if it’s already running.

```sh
pnpm alchemy deploy
curl -X PUT "$URL/users/ana/files/notes.md" -d 'hello'
curl "$URL/users/ana/files/notes.md"
```

## Next

[Part 4](part-4.md) changes the snapshot.
