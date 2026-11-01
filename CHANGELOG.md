# Changelog

## 0.1.1

- A store keeps the transport it is handed. Both `S3ObjectStore` constructors forwarded the transport to their shared initializer as a lend, so the store recorded a borrow and the constructor freed the transport when it returned. Every store from `S3ObjectStore.open`, and so every R2 store from cajeta-cloud-cloudflare's `R2ObjectStore.connect`, built its first request from freed memory. Tests over `FakeS3` passed because the test kept the fake alive. `S3OpenOwnershipTest` covers both constructors.

## 0.1.0 (2026-11-01)

- The S3 protocol client, moved out of cajeta-cloud-aws (its 3b95314, f2c17a3, 124b0b2): SigV4 (AWS test-suite vectors), the S3 wire layer, `S3ObjectStore` with conditional writes, listing, range reads, streaming and multipart. The endpoint and signing region are an `S3Endpoint`; credentials arrive in an `S3Config`; an `S3Profile` declares what a service lacks and the store refuses it by name. `FakeS3` ships in `dev.cajeta.cloud.s3.testkit` for provider tests.
- A profile carries a service's own error codes (`S3Profile.errorKind`, consulted by `S3Errors.of` before S3's table) and the store keeps an owned copy of its profile. `FakeS3.partRule(min, uniform)` makes the fake refuse a multipart upload that breaks a service's part rule, with `EntityTooSmall` or `InvalidPart` at complete.
