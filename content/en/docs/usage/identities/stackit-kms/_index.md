---
title: "STACKIT KMS"
weight: 80
description: You can use STACKIT's KMS to encrypt data.
---

## Encrypting using STACKIT KMS

The STACKIT KMS integration uses the
[STACKIT SDK for Go](https://github.com/stackitcloud/stackit-sdk-go)
default credential provider chain which tries several authentication methods, in
this order:

1. Static token or key flow credentials
2. Environment variable `STACKIT_SERVICE_ACCOUNT_TOKEN`
3. Credentials file at `~/.stackit/credentials.json`
4. Token flow via service account key

For more details, see the
[STACKIT KMS documentation](https://docs.stackit.cloud/products/security/kms/).

STACKIT KMS uses a resource ID in the format
`projects/<projectId>/regions/<regionId>/keyRings/<keyRingId>/keys/<keyId>/versions/<versionNumber>`.

You can list your KMS keys using the STACKIT CLI:

``` sh
stackit kms key-ring list --project-id PROJECT_ID --region eu01
stackit kms key list --project-id PROJECT_ID --region eu01 --key-ring-id KEYRING_ID
stackit kms key version list --project-id PROJECT_ID --region eu01 --key-ring-id KEYRING_ID --key-id KEY_ID
```

Now you can encrypt a file using:

``` sh
$ sops encrypt --stackit-kms projects/my-project-id/regions/eu01/keyRings/aaaaaaaa-1111-2222-3333-bbbbbbbbbbbb/keys/cccccccc-4444-5555-6666-dddddddddddd/versions/1 test.yaml > test.enc.yaml
```

Or using the environment variable:

``` sh
$ export SOPS_STACKIT_KMS_IDS="projects/my-project-id/regions/eu01/keyRings/aaaaaaaa-1111-2222-3333-bbbbbbbbbbbb/keys/cccccccc-4444-5555-6666-dddddddddddd/versions/1"
$ sops encrypt test.yaml > test.enc.yaml
```

And decrypt it using:

``` sh
$ sops decrypt test.enc.yaml
```

You can also configure STACKIT KMS keys in the `.sops.yaml` config file:

``` yaml
creation_rules:
    - path_regex: \.stackit\.yaml$
      stackit_kms: projects/my-project-id/regions/eu01/keyRings/aaaaaaaa-1111-2222-3333-bbbbbbbbbbbb/keys/cccccccc-4444-5555-6666-dddddddddddd/versions/1
```
