---
title: "Common operations"
weight: 30
---

## Examples

Take a look into the [examples
folder](https://github.com/getsops/sops/tree/main/examples) for detailed
use cases of SOPS in a CI environment. The section below describes
specific tips for common use cases.

## Saving Output to a File

By default SOPS just dumps all the output to the standard output. We can
use the `--output` flag followed by a filename to save the output to the
file specified. Beware using both `--in-place` and `--output` flags will
result in an error.

## Creating a new file

The command below creates a new file with a data key encrypted by KMS
and PGP.

``` sh
$ sops edit --kms "arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500" --pgp C9CAB0AF1165060DB58D6D6B2653B624D620786D /path/to/new/file.yaml
```

## Encrypting an existing file

Similar to the previous command, we tell SOPS to use one KMS and one PGP
key. The path points to an existing cleartext file, so we give `sops`
the flag `-e` to encrypt the file, and redirect the output to a
destination file.

``` sh
$ export SOPS_KMS_ARN="arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500"
$ export SOPS_PGP_FP="C9CAB0AF1165060DB58D6D6B2653B624D620786D"
$ sops encrypt /path/to/existing/file.yaml > /path/to/new/encrypted/file.yaml
```

Decrypt the file with `-d`.

``` sh
$ sops decrypt /path/to/new/encrypted/file.yaml
```

## Encrypt or decrypt a file in place

Rather than redirecting the output of `-e` or `-d`, `sops` can replace
the original file after encrypting or decrypting it.

``` sh
# file.yaml is in cleartext
$ sops encrypt -i /path/to/existing/file.yaml
# file.yaml is now encrypted
$ sops decrypt -i /path/to/existing/file.yaml
# file.yaml is back in cleartext
```

## Encrypting binary files

SOPS primary use case is encrypting YAML, JSON, ENV, and INI configuration files,
but it also has the ability to manage binary files. When encrypting a
binary, SOPS will read the data as bytes, encrypt it, store the
encrypted base64 under `tree['data']` and write the result as JSON.

Note that the base64 encoding of encrypted data can actually make the
encrypted file larger than the cleartext one.

In-place encryption/decryption also works on binary files.

``` sh
$ dd if=/dev/urandom of=/tmp/somerandom bs=1024
count=512
512+0 records in
512+0 records out
524288 bytes (524 kB) copied, 0.0466158 s, 11.2 MB/s

$ sha512sum /tmp/somerandom
9589bb20280e9d381f7a192000498c994e921b3cdb11d2ef5a986578dc2239a340b25ef30691bac72bdb14028270828dad7e8bd31e274af9828c40d216e60cbe /tmp/somerandom

$ sops encrypt -i /tmp/somerandom
please wait while a data encryption key is being generated and stored securely

$ sops decrypt -i /tmp/somerandom

$ sha512sum /tmp/somerandom
9589bb20280e9d381f7a192000498c994e921b3cdb11d2ef5a986578dc2239a340b25ef30691bac72bdb14028270828dad7e8bd31e274af9828c40d216e60cbe /tmp/somerandom
```

## Extract a sub-part of a document tree

SOPS can extract a specific part of a YAML or JSON document, by provided
the path in the `--extract` command line flag. This is useful to extract
specific values, like keys, without needing an extra parser.

``` sh
$ sops decrypt --extract '["app2"]["key"]' ~/git/svc/sops/example.yaml
-----BEGIN RSA PRIVATE KEY-----
MIIBPAIBAAJBAPTMNIyHuZtpLYc7VsHQtwOkWYobkUblmHWRmbXzlAX6K8tMf3Wf
ImcbNkqAKnELzFAPSBeEMhrBN0PyOC9lYlMCAwEAAQJBALXD4sjuBn1E7Y9aGiMz
bJEBuZJ4wbhYxomVoQKfaCu+kH80uLFZKoSz85/ySauWE8LgZcMLIBoiXNhDKfQL
vHECIQD6tCG9NMFWor69kgbX8vK5Y+QL+kRq+9HK6yZ9a+hsLQIhAPn4Ie6HGTjw
fHSTXWZpGSan7NwTkIu4U5q2SlLjcZh/AiEA78NYRRBwGwAYNUqzutGBqyXKUl4u
Erb0xAEyVV7e8J0CIQC8VBY8f8yg+Y7Kxbw4zDYGyb3KkXL10YorpeuZR4LuQQIg
bKGPkMM4w5blyE1tqGN0T7sJwEx+EUOgacRNqM2ljVA=
-----END RSA PRIVATE KEY-----
```

The tree path syntax uses regular python dictionary syntax, without the
variable name. Extract keys by naming them, and array elements by
numbering them.

``` sh
$ sops decrypt --extract '["an_array"][1]' ~/git/svc/sops/example.yaml
secretuser2
```

## Set a sub-part in a document tree

SOPS can set a specific part of a YAML or JSON document, by providing
the path and value in the `set` command. This is useful to set specific
values, like keys, without needing an editor.

``` sh
$ sops set ~/git/svc/sops/example.yaml '["app2"]["key"]' '"app2keystringvalue"'
```

The tree path syntax uses regular python dictionary syntax, without the
variable name. Set to keys by naming them, and array elements by
numbering them.

``` sh
$ sops set ~/git/svc/sops/example.yaml '["an_array"][1]' '"secretuser2"'
```

The value must be formatted as json.

``` sh
$ sops set ~/git/svc/sops/example.yaml '["an_array"][1]' '{"uid1":null,"uid2":1000,"uid3":["bob"]}'
```

You can also provide the value from a file or stdin:

``` sh
# Provide the value from a file
$ echo '{"uid1":null,"uid2":1000,"uid3":["bob"]}' > /tmp/example-value
$ sops set --value-file ~/git/svc/sops/example.yaml '["an_array"][1]' /tmp/example-value

# Provide the value from stdin
$ echo '{"uid1":null,"uid2":1000,"uid3":["bob"]}' | sops set --value-stdin ~/git/svc/sops/example.yaml '["an_array"][1]'
```

## Unset a sub-part in a document tree

Symmetrically, SOPS can unset a specific part of a YAML or JSON document, by providing
the path in the `unset` command. This is useful to unset specific values, like keys, without
needing an editor.

``` sh
$ sops unset ~/git/svc/sops/example.yaml '["app2"]["key"]'
```

The tree path syntax uses regular python dictionary syntax, without the
variable name. Set to keys by naming them, and array elements by
numbering them.

``` sh
$ sops unset ~/git/svc/sops/example.yaml '["an_array"][1]'
```

## Showing diffs in cleartext in git

You most likely want to store encrypted files in a version controlled
repository. SOPS can be used with git to decrypt files when showing
diffs between versions. This is very handy for reviewing changes or
visualizing history.

To configure SOPS to decrypt files during diff, create a
`.gitattributes` file at the root of your repository that contains a
filter and a command.

``` text
*.yaml diff=sopsdiffer
```

Here we only care about YAML files. `sopsdiffer` is an arbitrary name
that we map to a SOPS command in the git configuration file of the
repository.

``` sh
$ git config diff.sopsdiffer.textconv "sops decrypt"

$ grep -A 1 sopsdiffer .git/config
[diff "sopsdiffer"]
    textconv = "sops decrypt"
```

With this in place, calls to `git diff` will decrypt both previous and
current versions of the target file prior to displaying the diff. And it
even works with git client interfaces, because they call git diff under
the hood!

## Encrypting only parts of a file

> 📝 **Note**
>
> This only works on YAML, JSON, ENV, and INI files, not on BINARY files.

By default, SOPS encrypts all the values of a YAML, JSON, ENV, or INI file and
leaves the keys in cleartext. In some instances, you may want to exclude
some values from being encrypted. This can be accomplished by adding the
suffix **\_unencrypted** to any key of a file. When set, all values
underneath the key that set the **\_unencrypted** suffix will be left in
cleartext.

Note that, while in cleartext, unencrypted content is still added to the
checksum of the file, and thus cannot be modified outside of SOPS
without breaking the file integrity check. This behavior can be modified
using `--mac-only-encrypted` flag or `.sops.yaml` config file which
makes SOPS compute a MAC only over values it encrypted and not all
values.

The unencrypted suffix can be set to a different value using the
`--unencrypted-suffix` option.

Conversely, you can opt in to only encrypt some values in a YAML or JSON
file, by adding a chosen suffix to those keys and passing it to the
`--encrypted-suffix` option.

A third method is to use the `--encrypted-regex` which will only encrypt
values under keys that match the supplied regular expression. For
example, this command:

``` sh
$ sops encrypt --encrypted-regex '^(data|stringData)$' k8s-secrets.yaml
```

will encrypt the values under the `data` and `stringData` keys in a YAML
file containing kubernetes secrets. It will not encrypt other values
that help you to navigate the file, like `metadata` which contains the
secrets\' names.

Conversely, you can opt in to only leave certain keys without encrypting
by using the `--unencrypted-regex` option, which will leave the values
unencrypted of those keys that match the supplied regular expression.
For example, this command:

``` sh
$ sops encrypt --unencrypted-regex '^(description|metadata)$' k8s-secrets.yaml
```

will not encrypt the values under the `description` and `metadata` keys
in a YAML file containing kubernetes secrets, while encrypting
everything else.

For YAML files, another method is to use `--encrypted-comment-regex` which will
only encrypt comments and values which have a preceding comment matching the supplied
regular expression.

Conversely, you can opt in to only left certain keys without encrypting by using the
`--unencrypted-comment-regex` option, which will leave the values and comments
unencrypted when they have a preeceding comment, or a trailing comment on the same line,
that matches the supplied regular expression.

You can also specify these options in the `.sops.yaml` config file.

> 📝 **Note**
>
> These six options `--unencrypted-suffix`, `--encrypted-suffix`,
> `--encrypted-regex`, `--unencrypted-regex`, `--encrypted-comment-regex`,
> and `--unencrypted-comment-regex` are mutually exclusive and
> cannot all be used in the same file.
