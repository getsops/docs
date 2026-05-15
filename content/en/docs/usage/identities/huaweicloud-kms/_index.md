---
title: "HuaweiCloud KMS"
weight: 70
---

## Encrypting using HuaweiCloud KMS

The HuaweiCloud KMS integration uses the
[default credential provider chain](https://github.com/huaweicloud/huaweicloud-sdk-go-v3/blob/master/core/auth/provider/provider.go)
which tries several authentication methods, in this order:

1. Environment variables: `HUAWEICLOUD_SDK_AK`, `HUAWEICLOUD_SDK_SK`, `HUAWEICLOUD_SDK_PROJECT_ID`
2. Credentials file at `~/.huaweicloud/credentials`
3. Instance metadata (when running on HuaweiCloud instances)

For example, you can use environment variables:

``` bash
export HUAWEICLOUD_SDK_AK="your-access-key"
export HUAWEICLOUD_SDK_SK="your-secret-key"
export HUAWEICLOUD_SDK_PROJECT_ID="your-project-id"
```

Alternatively, you can create a credentials file at `~/.huaweicloud/credentials`:

``` sh
$ cat ~/.huaweicloud/credentials
[default]
ak = your-access-key
sk = your-secret-key
project_id = your-project-id
```

Encrypting/decrypting with HuaweiCloud KMS requires a KMS key ID in the format
`region:key-uuid`. You can get the key ID from the HuaweiCloud console or using
the HuaweiCloud API. The key ID format is `region:key-uuid` where:

- `region` is the HuaweiCloud region (e.g., `tr-west-1`, `cn-north-1`)
- `key-uuid` is the UUID of the KMS key (e.g., `abc12345-6789-0123-4567-890123456789`)

Now you can encrypt a file using:

``` sh
$ sops encrypt --hckms tr-west-1:abc12345-6789-0123-4567-890123456789 test.yaml > test.enc.yaml
```

Or using the environment variable:

``` sh
$ export SOPS_HUAWEICLOUD_KMS_IDS="tr-west-1:abc12345-6789-0123-4567-890123456789"
$ sops encrypt test.yaml > test.enc.yaml
```

And decrypt it using:

``` sh
$ sops decrypt test.enc.yaml
```

You can also configure HuaweiCloud KMS keys in the `.sops.yaml` config file:

``` yaml
creation_rules:
    - path_regex: \.hckms\.yaml$
      hckms:
        - tr-west-1:abc12345-6789-0123-4567-890123456789,tr-west-2:def67890-1234-5678-9012-345678901234
```
