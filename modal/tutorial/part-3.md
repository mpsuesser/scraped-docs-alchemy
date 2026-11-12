---
url: https://alchemy.run/modal/tutorial/part-3
title: "Part 3: A Copy per User"
description: "Give each user a Modal volume of their own, mounted into every sandbox they start from the warm image."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Modal sandbox lasts as long as its command, so a user’s files can’t live in it. They live on the user’s volume, and every sandbox the user starts from `Warm` mounts it: the warm image, plus their own files on top.

## Write a user’s files to their volume

```typescript
const match = /^\/users\/([a-z0-9-]+)\/files\/(.+)$/.exec(url.pathname);
if (match !== null) {
  const home = yield* volumes.getOrCreate(\`home-${match[1]}\`);
  if (request.method === "PUT") yield* home.writeFile(\`/${match[2]}\`, yield* request.text);
  return yield* HttpServerResponse.json({ content: yield* home.readFileString(\`/${match[2]}\`) });
}
```

The Worker writes the file straight to the volume. No sandbox runs.

## Every sandbox they start sees them

The `run` function from Part 2 already mounts the user’s volume at `/home/user`, so their next command reads what they wrote:

```sh
pnpm alchemy deploy
curl -X PUT "$URL/users/ada/files/notes.md" -d 'hello from ada'
curl -X POST "$URL/users/ada/run" -d 'ls && cat /home/user/notes.md'
```

The output lists the repository’s files from `Warm`, then `hello from ada` from Ada’s volume. Another user’s sandbox mounts their own volume and never sees hers.

## Delete a user’s files

```typescript
const user = /^\/users\/([a-z0-9-]+)$/.exec(url.pathname)?.[1];
if (request.method === "DELETE" && user !== undefined) {
  yield* volumes.delete(\`home-${user}\`);
  return HttpServerResponse.empty({ status: 204 });
}
```

Volumes your Worker creates aren’t deleted with the Stack until its environment is, so delete them when their owner goes away.

## Next

[Part 4](part-4.md) changes the image.
