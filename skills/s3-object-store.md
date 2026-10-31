---
id: s3-object-store
applies-to: [dev/cajeta/cloud/s3/S3ObjectStore, dev/cajeta/cloud/s3/S3Config, dev/cajeta/cloud/s3/S3Endpoint, dev/cajeta/cloud/s3/S3Profile, dev/cajeta/cloud/s3/HttpS3Transport, dev/cajeta/cloud/s3/S3Errors]
title: An S3-compatible store behind the object-store port — endpoints, profiles, conditional writes, streaming and multipart
description: Build an S3ObjectStore for a bucket on AWS S3, Cloudflare R2 or any S3-compatible endpoint from an S3Config, declare what the service lacks with an S3Profile, and know how conditional writes, listing, streaming reads and writes, multipart uploads and errors behave.
---

# The S3 protocol client

`dev.cajeta.cloud.s3.S3ObjectStore` implements cajeta-cloud's `ObjectStore`
over the S3 REST API. One implementation serves AWS S3, Cloudflare R2 and other
S3-compatible stores; the endpoint, the signing region and the provider's
profile are configuration. Code written against the port runs unchanged.

This library takes credentials outright and reads no environment: the
provider artifact that resolved the key (`cajeta-cloud-aws`'s chain,
`cajeta-cloud-cloudflare`'s configuration) builds the `S3Config`.

## Open a store

```cajeta
import dev.cajeta.cloud.s3.S3Config;
import dev.cajeta.cloud.s3.S3Credentials;
import dev.cajeta.cloud.s3.S3Endpoint;
import dev.cajeta.cloud.s3.S3ObjectStore;

S3Credentials creds = heap S3Credentials(keyId, secret, null);   // a session token for a temporary key

// AWS S3 in a region, virtual-host style
S3Endpoint aws #= S3Endpoint.aws("eu-west-2");
S3Config c = heap S3Config(#aws, "my-bucket", #creds);
S3ObjectStore store #= S3ObjectStore.open(c);

// An S3-compatible service: its base URL, the bucket in the path, its signing region
S3Endpoint r2 #= S3Endpoint.at("https://<account>.r2.cloudflarestorage.com", "auto", true);
```

`S3Config.partSize(bytes)` sets the part streamed writes and reads use (8 MiB
by default); `open` refuses one under the profile's minimum. `open` itself
calls nothing.

## Profiles

An `S3Profile` names what a service lacks. `S3Profile.aws()` excludes
nothing. A provider builds its own:

```cajeta
S3Profile p = heap S3Profile("r2");
p.exclude(Caps.SERVER_SIDE_COPY(), "CopyObject is not offered");
p.partRule(5242880L, true);    // every part but the last at least 5 MiB, and all the same size
S3ObjectStore store #= S3ObjectStore.open(c, p);
```

The store's `capabilities()` then declare only what the profile keeps, under
the profile's name, and an excluded operation throws `CloudException`
(`PROVIDER_ERROR`) naming the capability and the adapter before any request
is made (parent spec §3.2). Errors name the profile too (`'r2'`, not `'s3'`).

## Behaviour that matters

- **Conditional writes** are S3's: `putIfAbsent` sends `If-None-Match: *`,
  `putIfMatch` sends `If-Match: <etag>`. A lost condition returns
  `CondResult(won = false, etag = current)`, the current etag read with a HEAD
  (`""` when the key is absent). A 409 `ConditionalRequestConflict` (a racing
  conditional write) is retried three times, then thrown `TRANSIENT`.
- **Etags** are the service's, quotes included. Pass them back unchanged.
- **Listing** is ListObjectsV2, UTF-8 binary order. The page token is the
  continuation token; `""` on the last page.
- **Streaming**: `openWrite` holds one part and becomes a multipart upload
  only when a second part starts, so an object of any size costs one part of
  memory; `openRead` pulls one part-sized range at a time. At 8 MiB parts the
  writer reaches 78 GiB (10,000 parts). The writer's parts are all the same
  size but the last, so a uniform-parts profile holds.
- **Multipart** through `beginMultipart`: parts in any order, a re-upload
  replaces. Every part but the last must meet the profile's minimum
  (`EntityTooSmall` otherwise). An aborted upload leaves nothing readable; a
  bucket lifecycle rule cleans up uploads a crashed process never aborted.
- **delete** issues HEAD then DELETE, because S3 answers 204 either way.
- **getRange** clamps at the end of the object; an offset at or past the end
  throws `PROVIDER_ERROR` (416).
- **copy** is CopyObject, server side, when the profile keeps it.

## Errors

The `<Error><Code>` maps to `CloudException` kinds: `NoSuchKey`/`NoSuchUpload`
→ `NOT_FOUND`; `AccessDenied`, `InvalidAccessKeyId`, `SignatureDoesNotMatch`,
`ExpiredToken` → `ACCESS_DENIED`; `PreconditionFailed` →
`PRECONDITION_FAILED`; `SlowDown`, `ServiceUnavailable`, `InternalError`,
`RequestTimeout` → `TRANSIENT`; `NoSuchBucket` and the rest →
`PROVIDER_ERROR`. A HEAD has no body, so its status alone decides.

## Testing without a service

`heap S3ObjectStore(transport, bucket, partSize, profile)` takes any
`S3Transport`. `dev.cajeta.cloud.s3.testkit.FakeS3` is an in-memory bucket
answering in S3's documented shapes, shipped in this archive so a provider's
tests can drive their adapter through it; `FakeS3.log` records every request
and `conflicts` makes it answer that many conditional puts with a 409 first.
SigV4 is proven here against AWS's published test-suite vectors.
