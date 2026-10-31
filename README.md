# cajeta-cloud-s3

The S3 protocol client behind [`dev.cajeta.cloud`](https://github.com/jklappenbach/cajeta-cloud)'s
object-store port, written in Cajeta: AWS Signature Version 4 over
[cajeta-http](https://github.com/jklappenbach/cajeta-http), the S3 REST wire
layer, and `S3ObjectStore`, with the endpoint and signing region configurable.
One implementation serves AWS S3, Cloudflare R2 and other S3-compatible stores.

| Use | Where |
|---|---|
| AWS S3 with the AWS credential chain | [cajeta-cloud-aws](https://github.com/jklappenbach/cajeta-cloud-aws), `AwsS3.connect(bucket, ...)` |
| Cloudflare R2 | cajeta-cloud-cloudflare, `R2ObjectStore.connect(bucket, config)` |
| Any S3-compatible endpoint, credentials in hand | `S3ObjectStore.open(S3Config)` here |

An `S3Profile` declares what a service lacks (a capability, a part rule); the
store declares only what the profile keeps and refuses the rest by name. The
skill `s3-object-store` covers the behaviour.

**Status: in development (0.1.0 unreleased).** The plan is
`cajeta-cloud-cloudflare` Unit 1 in the cajeta repository's `agents/`.

## Build and test

```sh
./run-tests.sh
```

The suite needs no account: SigV4 is proven against AWS's published test-suite
vectors, and the store against `FakeS3`, an in-memory bucket answering in
S3's documented shapes, with the port's conformance suite run over it.
Dependency archives resolve from `~/.olla` or Olla at the versions
`cajeta.json` pins; `CLOUD_CJA=`, `HTTP_CJA=` and the like override one with a
local build.
