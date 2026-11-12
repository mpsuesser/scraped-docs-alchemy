---
url: https://alchemy.run/archil/storage/your-own-bucket
title: "Your own bucket"
description: "Keep a disk's files as ordinary objects in your own S3, R2, GCS or Azure bucket, with no keys for S3."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A disk can keep its files in a bucket you own instead of Archil’s. Archil still keeps the data in use in its storage layer, and copies writes to your bucket in the background, where they’re ordinary objects any S3 tool can read without Archil. Destroying the disk leaves them there. Archil offers it on its paid plans.

## Your bucket trusts Archil, no key needed

On S3, the bucket’s policy trusts Archil’s IAM role for your region, only on behalf of disks in your organization, so no access key exists anywhere.

<svg viewBox="0 0 501 274" style="max-width: 576px" role="img" aria-label="The bucket policy lets Archil's role in only for disks in your organization. A disk in another organization, using the same role, is refused."><defs><marker id="alc-bd-arrow-lzuqbn" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="286.4" y="118" width="198.60000000000002" height="140" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="385.7" y1="64" x2="385.7" y2="144" marker-end="url(#alc-bd-arrow-lzuqbn)"></line><text fill="currentColor" x="395.7" y="110">Archil's role</text></g> <g><path stroke="currentColor" fill="none" d="M 97.3 196 V 242 H 282.4"></path><path stroke="currentColor" fill="none" d="M 277.4 235 l 12 14 m 0 -14 l -12 14"></path></g><text fill="currentColor" x="473" y="137" text-anchor="end">AWS account</text> <g><rect stroke="currentColor" fill="none" x="304.4" y="16" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="385.7" y="37" text-anchor="middle">Dataset</text> <text fill="currentColor" x="385.7" y="53" text-anchor="middle">disk · org acme</text></g> <g><rect stroke="currentColor" fill="none" x="304.4" y="148" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="385.7" y="169" text-anchor="middle">Store</text> <text fill="currentColor" x="385.7" y="185" text-anchor="middle">S3 Bucket · datasets/</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="97.3" y="169" text-anchor="middle">notes</text> <text fill="currentColor" x="97.3" y="185" text-anchor="middle">disk · another org</text></g></svg> <svg viewBox="0 0 246.6 394" style="max-width: 284px" role="img" aria-label="The bucket policy lets Archil's role in only for disks in your organization. A disk in another organization, using the same role, is refused."><defs><marker id="alc-bd-arrow-tskslm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="282" width="198.6" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="115.3" y1="212" x2="115.3" y2="308" marker-end="url(#alc-bd-arrow-tskslm)"></line><text fill="currentColor" x="125.3" y="266">Archil's role</text></g> <g><path stroke="currentColor" fill="none" d="M 196.6 40 H 230.6 V 304 H 218.6"></path><path stroke="currentColor" fill="none" d="M 207.6 297 l 14 14 m 0 -14 l -14 14"></path></g><text fill="currentColor" x="202.6" y="301" text-anchor="end">AWS account</text> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="185" text-anchor="middle">Dataset</text> <text fill="currentColor" x="115.3" y="201" text-anchor="middle">disk · org acme</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="333" text-anchor="middle">Store</text> <text fill="currentColor" x="115.3" y="349" text-anchor="middle">S3 Bucket · datasets/</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="162.6" height="48" rx="8"></rect><text fill="currentColor" x="115.3" y="37" text-anchor="middle">notes</text> <text fill="currentColor" x="115.3" y="53" text-anchor="middle">disk · another org</text></g></svg>

The bucket policy lets Archil's role in only for disks in your organization. A disk in another organization, using the same role, is refused.

## A disk on an S3 bucket

`ArchilAWS.S3Disk` declares the bucket policy and the disk together:

```typescript
import * as AWS from "@alchemy.run/aws";
import * as ArchilAWS from "@alchemy.run/archil/AWS";

export const Store = AWS.S3.Bucket("Store", {});
export const Dataset = ArchilAWS.S3Disk("Dataset", { bucket: Store, prefix: "datasets/" });
```

The disk’s files land under `datasets/` in `Store`. Your Stack needs `AWS.providers()` next to `Archil.providers()`.

## Set your organization id in the profile

The policy is scoped to your Archil organization, so Alchemy needs its id. Enter it in your profile:

```sh
alchemy profile edit --add Archil
```

Or set `ARCHIL_ORGANIZATION` in CI. Find the id in the [console](https://console.archil.com/).

## Use another object store

R2, Google Cloud Storage, Azure Blob and other S3-compatible stores take keys. Give them in `storage` on `Archil.Disk`:

```typescript
export const Dataset = Archil.Disk("Dataset", {
  storage: {
    type: "r2",
    bucketName: "datasets",
    bucketEndpoint: "https://<account>.r2.cloudflarestorage.com",
    accessKeyId: Config.Redacted("R2_ACCESS_KEY_ID"),
    secretAccessKey: Config.Redacted("R2_SECRET_ACCESS_KEY"),
  },
});
```

Changing `storage` replaces the disk.

## The bucket sees writes a little later

Archil writes to the bucket after the write is durable in its own storage, so a tool reading the bucket directly sees a new file a little later than a client of the disk does. Read through the disk when you need the latest write.
