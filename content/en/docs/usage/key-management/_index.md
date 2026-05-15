---
title: "Key management"
weight: 40
---

## Adding and removing keys

When creating new files, `sops` uses the PGP, KMS and GCP KMS defined in
the command line arguments `--kms`, `--pgp`, `--gcp-kms`, `--hckms` or
`--azure-kv`, or from the environment variables `SOPS_KMS_ARN`,
`SOPS_PGP_FP`, `SOPS_GCP_KMS_IDS`, `SOPS_HUAWEICLOUD_KMS_IDS`, `SOPS_AZURE_KEYVAULT_URLS`. That
information is stored in the file under the `sops` section, such that
decrypting files does not require providing those parameters again.

Master PGP and KMS keys can be added and removed from a `sops` file in
one of three ways:

1.  By using a `.sops.yaml` file and the `updatekeys` command.
2.  By using command line flags.
3.  By editing the file directly.

The SOPS team recommends the `updatekeys` approach.

### `updatekeys` command

The `updatekeys` command uses the
[.sops.yaml](#using-sops-yaml-conf-to-select-kms-pgp-for-new-files)
configuration file to update (add or remove) the corresponding secrets
in the encrypted file. Note that the example below uses the [Block
Scalar yaml construct](https://yaml-multiline.info/) to build a space
separated list.

``` yaml
creation_rules:
    - pgp: >-
        85D77543B3D624B63CEA9E6DBC17301B491B3F21,
        FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4
```

``` sh
$ sops updatekeys test.enc.yaml
```

SOPS will prompt you with the changes to be made. This interactivity can
be disabled by supplying the `-y` flag.

### `rotate` command

The `rotate` command generates a new data encryption key and reencrypt
all values with the new key. At the same time, the command line flag
`--add-kms`, `--add-pgp`, `--add-gcp-kms`, `--add-hckms`, `--add-azure-kv`, `--rm-kms`,
`--rm-pgp`, `--rm-gcp-kms`, `--rm-hckms` and `--rm-azure-kv` can be used to add and
remove keys from a file. These flags use the comma separated syntax as
the `--kms`, `--pgp`, `--gcp-kms`, `--hckms` and `--azure-kv` arguments when
creating new files.

Use `updatekeys` if you want to add a key without rotating the data key.

``` sh
# add a new pgp key to the file and rotate the data key
$ sops rotate -i --add-pgp 85D77543B3D624B63CEA9E6DBC17301B491B3F21 example.yaml

# remove a pgp key from the file and rotate the data key
$ sops rotate -i --rm-pgp 85D77543B3D624B63CEA9E6DBC17301B491B3F21 example.yaml
```

### Direct Editing

Alternatively, invoking `sops edit` with the flag **-s** will display
the master keys while editing. This method can be used to add or remove
`kms` or `pgp` keys under the `sops` section.

For example, to add a KMS master key to a file, add the following entry
while editing:

``` yaml
sops:
    kms:
        - arn: arn:aws:kms:us-east-1:656532927350:key/920aff2e-c5f1-4040-943a-047fa387b27e
```

And, similarly, to add a PGP master key, we add its fingerprint:

``` yaml
sops:
    pgp:
        - fp: 85D77543B3D624B63CEA9E6DBC17301B491B3F21
```

When the file is saved, SOPS will update its metadata and encrypt the
data key with the freshly added master keys. The removed entries are
simply deleted from the file.

When removing keys, it is recommended to rotate the data key using `-r`,
otherwise, owners of the removed key may have add access to the data key
in the past.

## Key Rotation

It is recommended to renew the data key on a regular basis. `sops`
supports key rotation via the `rotate` command. Invoking it on an
existing file causes `sops` to reencrypt the file with a new data key,
which is then encrypted with the various KMS and PGP master keys defined
in the file.

Add the `-i` option to write the rotated file back, instead of printing
it to stdout.

``` sh
$ sops rotate example.yaml
```
