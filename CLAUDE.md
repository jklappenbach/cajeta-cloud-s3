# cajeta-cloud-s3

The S3 protocol client for dev.cajeta.cloud's object-store port (package
`dev.cajeta.cloud.s3`): SigV4, the wire layer, `S3ObjectStore`, and `FakeS3`
in `testkit`. Spec and plan: `specs/cajeta-cloud-cloudflare-spec.md` §2 and
`agents/cajeta-cloud-cloudflare-plan.md` Unit 1 in the cajeta repository;
this repo carries no specs of its own. Moved out of cajeta-cloud-aws on
2026-10-30 so `cajeta-cloud-aws` and `cajeta-cloud-cloudflare` share it.

## Build & test

```sh
./run-tests.sh      # resolves dependency archives, builds the .cja, runs @Test discovery
```

No test reaches a service. SigV4 runs against AWS's published vectors under
`src/test/resources/sigv4`; the store runs against `FakeS3`.

## Rules

- Test first: the failing test before the implementation, every change.
- This library reads no environment and no file: credentials arrive in an
  `S3Config`. A provider artifact owns its chain.
- Nothing logs a credential; `S3Credentials.toString()` names the key id only.
- A provider difference is a profile entry or a declared capability, never a
  branch on the endpoint's hostname.
- Releases ship from the `release` branch; CI's guard refuses a tag not on it.
