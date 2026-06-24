---
title: "PGP / GnuPG"
weight: 10
description: You can use PGP / GnuPG to encrypt data.
---

## Test with the dev PGP key

If you want to test **SOPS** without having to do a bunch of setup, you
can use the example files and pgp key provided with the repository:

``` sh
$ git clone https://github.com/getsops/sops.git
$ cd sops
$ gpg --import pgp/sops_functional_tests_key.asc
$ sops edit example.yaml
```

This last step will decrypt `example.yaml` using the test private key.

## Encrypting with GnuPG subkeys

If you want to encrypt with specific GnuPG subkeys, it does not suffice to provide the
exact key ID of the subkey to SOPS, since GnuPG might use *another* subkey instead
to encrypt the file key with. To force GnuPG to use a specific subkey, you need to
append `!` to the key's fingerprint.

``` yaml
creation_rules:
    - pgp: >-
        85D77543B3D624B63CEA9E6DBC17301B491B3F21!,
        E60892BB9BD89A69F759A1A0A3D652173B763E8F!
```

Please note that this is only passed on correctly to GnuPG since SOPS 3.9.3.

## Specify a different GPG executable

SOPS checks for the `SOPS_GPG_EXEC` environment variable. If specified,
it will attempt to use the executable set there instead of the default
of `gpg`.

Example: place the following in your `~/.bashrc`

``` bash
SOPS_GPG_EXEC = 'your_gpg_client_wrapper'
```
