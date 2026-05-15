---
title: "Key groups"
weight: 200
description: Key groups allow to require multiple identities together to edit or decrypt a file.
---

## Key groups

By default, SOPS encrypts the data key for a file with each of the
master keys, such that if any of the master keys is available, the file
can be decrypted. However, it is sometimes desirable to require access
to multiple master keys in order to decrypt files. This can be achieved
with key groups.

When using key groups in SOPS, data keys are split into parts such that
keys from multiple groups are required to decrypt a file. SOPS uses
Shamir\'s Secret Sharing to split the data key such that each key group
has a fragment, each key in the key group can decrypt that fragment, and
a configurable number of fragments (threshold) are needed to decrypt and
piece together the complete data key. When decrypting a file using
multiple key groups, SOPS goes through key groups in order, and in each
group, tries to recover the fragment of the data key using a master key
from that group. Once the fragment is recovered, SOPS moves on to the
next group, until enough fragments have been recovered to obtain the
complete data key.

By default, the threshold is set to the number of key groups. For
example, if you have three key groups configured in your SOPS file and
you don\'t override the default threshold, then one master key from each
of the three groups will be required to decrypt the file.

Management of key groups is done with the `sops groups` command.

For example, you can add a new key group with 3 PGP keys and 3 KMS keys
to the file `my_file.yaml`:

``` sh
$ sops groups add --file my_file.yaml --pgp fingerprint1 --pgp fingerprint2 --pgp fingerprint3 --kms arn1 --kms arn2 --kms arn3
```

Or you can delete the 1st group (group number 0, as groups are
zero-indexed) from `my_file.yaml`:

``` sh
$ sops groups delete --file my_file.yaml 0
```

Key groups can also be specified in the `.sops.yaml` config file, like
so:

``` yaml
creation_rules:
    - path_regex: .*keygroups.*
      key_groups:
          # First key group
          - pgp:
                - fingerprint1
                - fingerprint2
            kms:
                - arn: arn1
                  role: role1
                  context:
                      foo: bar
                - arn: arn2
                  aws_profile: myprofile
          # Second key group
          - pgp:
                - fingerprint3
                - fingerprint4
            kms:
                - arn: arn3
                - arn: arn4
          # Third key group
          - pgp:
                - fingerprint5
```

Given this configuration, we can create a new encrypted file like we
normally would, and optionally provide the
`--shamir-secret-sharing-threshold` command line flag if we want to
override the default threshold. SOPS will then split the data key into
three parts (from the number of key groups) and encrypt each fragment
with the master keys found in each group.

For example:

``` sh
$ sops edit --shamir-secret-sharing-threshold 2 example.json
```

Alternatively, you can configure the Shamir threshold for each creation
rule in the `.sops.yaml` config with `shamir_threshold`:

``` yaml
creation_rules:
    - path_regex: .*keygroups.*
      shamir_threshold: 2
      key_groups:
          # First key group
          - pgp:
                - fingerprint1
                - fingerprint2
            kms:
                - arn: arn1
                  role: role1
                  context:
                      foo: bar
                - arn: arn2
                  aws_profile: myprofile
          # Second key group
          - pgp:
                - fingerprint3
                - fingerprint4
            kms:
                - arn: arn3
                - arn: arn4
          # Third key group
          - pgp:
                - fingerprint5
```

And then run `sops edit example.json`.

The threshold (`shamir_threshold`) is set to 2, so this configuration
will require master keys from two of the three different key groups in
order to decrypt the file. You can then decrypt the file the same way as
with any other SOPS file:

``` sh
$ sops decrypt example.json
```
