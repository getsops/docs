---
title: "Age"
weight: 20
---

## Encrypting using age

[age](https://age-encryption.org/) is a simple, modern, and secure tool
for encrypting files. It\'s recommended to use age over PGP, if
possible.

You can encrypt a file for one or more age recipients (comma separated)
using the `--age` option or the **SOPS_AGE_RECIPIENTS** environment
variable:

``` sh
$ sops encrypt --age age1yt3tfqlfrwdwx0z0ynwplcr6qxcxfaqycuprpmy89nr83ltx74tqdpszlw test.yaml > test.enc.yaml
```

When decrypting a file with the corresponding identity, SOPS will look
for a text file name `keys.txt` located in a `sops` subdirectory of your
user configuration directory.

* **Linux**

  - Looks for `keys.txt` in `$XDG_CONFIG_HOME/sops/age/keys.txt`;
  - Falls back to `$HOME/.config/sops/age/keys.txt` if `$XDG_CONFIG_HOME` isn't set.

* **macOS**

  - Looks for `keys.txt` in `$XDG_CONFIG_HOME/sops/age/keys.txt`;
  - Falls back to `$HOME/Library/Application Support/sops/age/keys.txt` if `$XDG_CONFIG_HOME` isn't set.

* **Windows**
  - Looks for `keys.txt` in `%AppData%\\sops\\age\\keys.txt`.

You can override the default lookup by:

* setting the environment variable **SOPS_AGE_KEY_FILE**;
* setting the **SOPS_AGE_KEY** environment variable;
* providing a command to output the age keys by setting the **SOPS_AGE_KEY_CMD** environment variable.
  This command can read the age recipient for which to return the private key from the **SOPS_AGE_RECIPIENT** environment variable.

The contents of this key file should be a list of age X25519 identities,
one per line. Lines beginning with `#` are considered comments and
ignored. Each identity will be tried in sequence until one is able to
decrypt the data.

Encrypting with SSH keys via age is also supported by SOPS.
You can use SSH public keys (`ssh-ed25519 AAAA...`, `ssh-rsa AAAA...`)
as age recipients when encrypting a file.

When decrypting a file, SOPS will attempt to source the SSH private key as follows:

* From the path specified in environment variable **SOPS_AGE_SSH_PRIVATE_KEY_FILE**.
* From the output of the command specified in environment variable **SOPS_AGE_SSH_PRIVATE_KEY_CMD**.

  > 📝 **Note**
  >
  > The output of this command must provide a key that is not password protected.

* From `~/.ssh/id_ed25519`.
* From `~/.ssh/id_rsa`.

Note that only `ssh-rsa` and `ssh-ed25519` are supported.

A list of age recipients can be added to the `.sops.yaml`:

``` yaml
creation_rules:
    - age: >-
        age1s3cqcks5genc6ru8chl0hkkd04zmxvczsvdxq99ekffe4gmvjpzsedk23c,
        age1qe5lxzzeppw5k79vxn3872272sgy224g2nzqlzy3uljs84say3yqgvd0sw
```

It is also possible to use `updatekeys`, when adding or removing age recipients. For example:

``` sh
$ sops updatekeys secret.enc.yaml
2022/02/09 16:32:02 Syncing keys for file /iac/solution1/secret.enc.yaml
The following changes will be made to the file's groups:
Group 1
    age1s3cqcks5genc6ru8chl0hkkd04zmxvczsvdxq99ekffe4gmvjpzsedk23c
+++ age1qe5lxzzeppw5k79vxn3872272sgy224g2nzqlzy3uljs84say3yqgvd0sw
Is this okay? (y/n):y
2022/02/09 16:32:04 File /iac/solution1/secret.enc.yaml synced with new keys
```
