---
title: "Publishing"
weight: 50
---

## Using the publish command

`sops publish $file` publishes a file to a pre-configured destination
(this lives in the SOPS config file). Additionally, support
re-encryption rules that work just like the creation rules.

This command requires a `.sops.yaml` configuration file. Below is an
example:

``` yaml
destination_rules:
    - s3_bucket: "sops-secrets"
      path_regex: s3/*
      recreation_rule:
          pgp: F69E4901EDBAD2D1753F8C67A64535C4163FB307
    - gcs_bucket: "sops-secrets"
      path_regex: gcs/*
      recreation_rule:
          pgp: F69E4901EDBAD2D1753F8C67A64535C4163FB307
    - vault_path: "sops/"
      vault_kv_mount_name: "secret/" # default
      vault_kv_version: 2 # default
      path_regex: vault/*
      omit_extensions: true
```

The above configuration will place all files under `s3/*` into the S3
bucket `sops-secrets`, all files under `gcs/*` into the GCS bucket
`sops-secrets`, and the contents of all files under `vault/*` into
Vault\'s KV store under the path `secrets/sops/`. For the files that
will be published to S3 and GCS, it will decrypt them and re-encrypt
them using the `F69E4901EDBAD2D1753F8C67A64535C4163FB307` pgp key.

You would deploy a file to S3 with a command like:
`sops publish s3/app.yaml`

To publish all files in selected directory recursively, you need to
specify `--recursive` flag.

If you don\'t want file extension to appear in destination secret path,
use `--omit-extensions` flag or `omit_extensions: true` in the
destination rule in `.sops.yaml`.

### Publishing to Vault

There are a few settings for Vault that you can place in your
destination rules. The first is `vault_path`, which is required. The
others are optional, and they are `vault_address`,
`vault_kv_mount_name`, `vault_kv_version`.

SOPS uses the official Vault API provided by Hashicorp, which makes use
of [environment
variables](https://www.vaultproject.io/docs/commands/#environment-variables)
for configuring the client.

`vault_kv_mount_name` is used if your Vault KV is mounted somewhere
other than `secret/`. `vault_kv_version` supports `1` and `2`, with `2`
being the default.

If the destination secret path already exists in Vault and contains the
same data as the source file, it will be skipped.

Below is an example of publishing to Vault (using token auth with a
local dev instance of Vault).

``` sh
$ export VAULT_TOKEN=...
$ export VAULT_ADDR='http://127.0.0.1:8200'
$ sops decrypt vault/test.yaml
example_string: bar
example_number: 42
example_map:
    key: value
$ sops publish vault/test.yaml
uploading /home/user/sops_directory/vault/test.yaml to http://127.0.0.1:8200/v1/secret/data/sops/test.yaml ? (y/n): y
$ vault kv get secret/sops/test.yaml
====== Metadata ======
Key              Value
---              -----
created_time     2019-07-11T03:32:17.074792017Z
deletion_time    n/a
destroyed        false
version          3

========= Data =========
Key               Value
---               -----
example_map       map[key:value]
example_number    42
example_string    bar
```
