# Changelog

## 0.1.0 (unreleased)

- The S3 protocol client, moved out of cajeta-cloud-aws (its 3b95314, f2c17a3, 124b0b2): SigV4 (AWS test-suite vectors), the S3 wire layer, `S3ObjectStore` with conditional writes, listing, range reads, streaming and multipart. The endpoint and signing region are an `S3Endpoint`; credentials arrive in an `S3Config`; an `S3Profile` declares what a service lacks and the store refuses it by name. `FakeS3` ships in `dev.cajeta.cloud.s3.testkit` for provider tests.
