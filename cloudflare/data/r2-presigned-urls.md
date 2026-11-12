---
url: https://alchemy.run/cloudflare/data/r2-presigned-urls
title: "R2 presigned URLs"
description: "Mint presigned R2 URLs so browsers upload and download objects directly, with the same code in alchemy dev and in production, from Effect-native or async Workers."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A presigned URL lets a browser upload or download one object in an [R2 bucket](r2.md) directly, without credentials. Your Worker mints the URL, and the same code works deployed and under `alchemy dev`.

<svg viewBox="0 0 500 285" style="max-width: 575px" role="img" aria-label="The Worker only signs. The file goes from the browser to R2 directly, and the URL works for one object until it expires."><defs><marker id="alc-seq-arrow-qd3czm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-qd3czm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="59" x2="82" y2="281"></line><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="82" y="42.5" text-anchor="middle">browser</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="59" x2="250" y2="281"></line><rect stroke="currentColor" fill="none" x="184" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="250" y="36" text-anchor="middle">Api</text> <text fill="currentColor" x="250" y="51" text-anchor="middle">Worker</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="59" x2="418" y2="281"></line><rect stroke="currentColor" fill="none" x="352" y="16" width="132" height="43" rx="8"></rect><text fill="currentColor" x="418" y="36" text-anchor="middle">Uploads</text> <text fill="currentColor" x="418" y="51" text-anchor="middle">R2 bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="85" x2="244" y2="85" marker-end="url(#alc-seq-arrow-qd3czm)"></line><rect stroke="currentColor" fill="none" x="109.2" y="66" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="166" y="78" text-anchor="middle">POST /upload-url</text></g> <g><rect stroke="currentColor" x="148.9" y="113" width="202.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="250" y="129" text-anchor="middle">sign a PUT for one key, 60s</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="250" y1="161" x2="88" y2="161" marker-end="url(#alc-seq-arrow-qd3czm)"></line><rect stroke="currentColor" fill="none" x="119.1" y="142" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="166" y="154" text-anchor="middle">presigned URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="82" y1="201" x2="412" y2="201" marker-end="url(#alc-seq-arrow-qd3czm)"></line><rect stroke="currentColor" fill="none" x="153.60000000000002" y="182" width="192.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="250" y="194" text-anchor="middle">PUT the file, straight to R2</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="418" y1="241" x2="88" y2="241" marker-end="url(#alc-seq-arrow-qd3czm)"></line><rect stroke="currentColor" fill="none" x="236.1" y="222" width="27.799999999999997" height="15" rx="3"></rect><text fill="currentColor" x="250" y="234" text-anchor="middle">200</text></g></svg> <svg viewBox="0 0 322 271" style="max-width: 370px" role="img" aria-label="The Worker only signs. The file goes from the browser to R2 directly, and the URL works for one object until it expires."><defs><marker id="alc-seq-arrow-n-qd3czm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker><marker id="alc-seq-arrow-warn-n-qd3czm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><line stroke-opacity="0.2" stroke="currentColor" x1="37" y1="45" x2="37" y2="267"></line><rect stroke="currentColor" fill="none" x="2" y="2" width="70" height="43" rx="8"></rect><text fill="currentColor" x="37" y="28.5" text-anchor="middle">browser</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="45" x2="161" y2="267"></line><rect stroke="currentColor" fill="none" x="126" y="2" width="70" height="43" rx="8"></rect><text fill="currentColor" x="161" y="22" text-anchor="middle">Api</text> <text fill="currentColor" x="161" y="37" text-anchor="middle">Worker</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="285" y1="45" x2="285" y2="267"></line><rect stroke="currentColor" fill="none" x="250" y="2" width="70" height="43" rx="8"></rect><text fill="currentColor" x="285" y="22" text-anchor="middle">Uploads</text> <text fill="currentColor" x="285" y="37" text-anchor="middle">R2 bucket</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="37" y1="71" x2="155" y2="71" marker-end="url(#alc-seq-arrow-n-qd3czm)"></line><rect stroke="currentColor" fill="none" x="42.2" y="52" width="113.6" height="15" rx="3"></rect><text fill="currentColor" x="99" y="64" text-anchor="middle">POST /upload-url</text></g> <g><rect stroke="currentColor" x="59.900000000000006" y="99" width="202.2" height="24" rx="5" style="fill: Canvas"></rect><text fill="currentColor" x="161" y="115" text-anchor="middle">sign a PUT for one key, 60s</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="161" y1="147" x2="43" y2="147" marker-end="url(#alc-seq-arrow-n-qd3czm)"></line><rect stroke="currentColor" fill="none" x="52.1" y="128" width="93.8" height="15" rx="3"></rect><text fill="currentColor" x="99" y="140" text-anchor="middle">presigned URL</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="37" y1="187" x2="279" y2="187" marker-end="url(#alc-seq-arrow-n-qd3czm)"></line><rect stroke="currentColor" fill="none" x="64.60000000000001" y="168" width="192.79999999999998" height="15" rx="3"></rect><text fill="currentColor" x="161" y="180" text-anchor="middle">PUT the file, straight to R2</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="285" y1="227" x2="43" y2="227" marker-end="url(#alc-seq-arrow-n-qd3czm)"></line><rect stroke="currentColor" fill="none" x="147.1" y="208" width="27.799999999999997" height="15" rx="3"></rect><text fill="currentColor" x="161" y="220" text-anchor="middle">200</text></g></svg>

The Worker only signs. The file goes from the browser to R2 directly, and the URL works for one object until it expires.

## Effect-native Workers

Bind the bucket with `PresignPutObject` / `PresignGetObject` and provide their `*Token` layers:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Uploads } from "./uploads.ts";

export default Cloudflare.Worker(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const presignPut = yield* Cloudflare.R2.PresignPutObject(Uploads);
    const presignGet = yield* Cloudflare.R2.PresignGetObject(Uploads);

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const url = new URL(request.url, "http://worker");
        const key = url.searchParams.get("key")!;

        if (url.pathname === "/upload-url") {
          return yield* HttpServerResponse.json({
            url: yield* presignPut({ key, contentType: "image/png" }),
          });
        }
        return yield* HttpServerResponse.json({
          url: yield* presignGet({ key, expiresIn: 60 }),
        });
      }).pipe(Effect.orDie),
    };
  }).pipe(
    Effect.provide(
      Layer.mergeAll(
        Cloudflare.R2.PresignPutObjectToken,
        Cloudflare.R2.PresignGetObjectToken,
      ),
    ),
  ),
);
```

Deployed, Alchemy mints an API token scoped to the bucket for the Worker, and the URLs point at `https://{accountId}.r2.cloudflarestorage.com`. Under `alchemy dev` they point at the Worker’s local R2 endpoint, `http://localhost:{port}/cdn-cgi/local/r2/s3`. Signing happens inside the Worker, so minting a URL makes no network request.

The browser uploads with a plain `fetch`, sending the `Content-Type` that was signed into the URL:

```typescript
const { url } = await (await fetch(\`/upload-url?key=${key}\`)).json();
await fetch(url, {
  method: "PUT",
  body: file,
  headers: { "content-type": "image/png" },
});
```

Both functions accept a few options:

```typescript
yield* presignPut({
  key: "avatars/me.png",
  expiresIn: 300, // seconds, 900 by default, 7 days at most
  contentType: "image/png", // the uploader must send this header
});

yield* presignGet({
  key: "reports/2026-09.pdf",
  expiresIn: 60,
  contentType: "application/pdf", // served with this Content-Type
  contentDisposition: 'attachment; filename="report.pdf"', // force a download
});
```

## Async Workers

Async Workers sign URLs themselves with an S3 client, which needs S3 credentials. `S3Credentials` is the easy way to get them: it creates an API token scoped to the permissions you ask for and hands the Worker its S3 keys. Declare it on `env`:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import { Uploads } from "./src/uploads.ts";

export const Api = Cloudflare.Worker("Api", {
  main: "./src/worker.ts",
  env: {
    UPLOADS_S3: Cloudflare.R2.S3Credentials(Uploads, { access: "write" }),
  },
});

export type ApiEnv = Cloudflare.InferEnv<typeof Api>;
```

Then sign with any S3 client. This one uses [`aws4fetch`](https://github.com/mhart/aws4fetch):

```typescript
import { AwsClient } from "aws4fetch";
import type * as Cloudflare from "@alchemy.run/cloudflare";
import type { ApiEnv } from "../alchemy.run.ts";

export default {
  async fetch(request: Request, env: ApiEnv) {
    const key = new URL(request.url).searchParams.get("key")!;
    const s3: Cloudflare.R2.S3CredentialsValue = JSON.parse(env.UPLOADS_S3);
    const client = new AwsClient({ ...s3, service: "s3" });

    const path = key.split("/").map(encodeURIComponent).join("/");
    const url = new URL(
      \`${s3.endpoint}/${encodeURIComponent(s3.bucketName)}/${path}\`,
    );
    url.searchParams.set("X-Amz-Expires", "900");
    const signed = await client.sign(url.toString(), {
      method: "PUT",
      headers: { "content-type": "image/png" },
      aws: { signQuery: true, allHeaders: true },
    });

    return Response.json({ url: signed.url });
  },
};
```

`UPLOADS_S3` is a JSON string holding `endpoint`, `bucketName`, `region`, `accessKeyId` and `secretAccessKey`. Deployed, it’s a secret. Under `alchemy dev`, no token is created and it points at the local endpoint with local credentials.

`access` is required and sets the token’s permissions:

```typescript
Cloudflare.R2.S3Credentials(Uploads, { access: "read" }); // downloads only
Cloudflare.R2.S3Credentials(Uploads, { access: "write" }); // uploads only
Cloudflare.R2.S3Credentials(Uploads, { access: "read-write" }); // both
```

In an Effect-native Worker, `yield* Cloudflare.R2.S3Credentials(Uploads, { access })` gives you the same values for your own S3 client.

## Browser uploads to a deployed bucket

A browser calling the URL from another origin needs a CORS rule on the bucket. The local endpoint allows any origin and ignores these rules, so a missing origin or header only shows up once deployed.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Uploads = Cloudflare.R2.Bucket("Uploads", {
  cors: [
    {
      allowedMethods: ["GET", "PUT"],
      allowedOrigins: ["https://app.example.com"],
      allowedHeaders: ["content-type"],
    },
  ],
});
```

## Local development

The local endpoint rejects expired and tampered URLs like R2 does, and objects uploaded through it are visible to the Worker’s own R2 bindings. S3 clients such as the AWS SDK work against it too. A bucket piped through `Alchemy.remote()` uses real R2 in dev.

Mint presigned URLs on demand and store object keys, not URLs. URLs expire, and a local URL points at the Worker’s dev port, which can change between `alchemy dev` runs.

R2 doesn’t support presigned POST (HTML form) uploads, locally or deployed. Use presigned `PUT` URLs.

## Where next

- [Local development](../local-development.md)
- [R2 API reference](https://alchemy.run/providers/cloudflare/r2)
