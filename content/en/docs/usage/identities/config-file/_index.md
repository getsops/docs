---
title: "Config file"
weight: 210
description: How to use `.sops.yaml` config files to select which identities to use for new files.
---

## Using .sops.yaml conf to select KMS, PGP and age for new files

It is often tedious to specify the `--kms` `--gcp-kms` `--hckms` `--pgp` and
`--age` parameters for creation of all new files. If your secrets are
stored under a specific directory, like a `git` repository, you can
create a `.sops.yaml` configuration file at the root directory to define
which keys are used for which filename.

> 📝 **Note**
>
> The file needs to be named `.sops.yaml`. Other names (i.e. `.sops.yml`) won't be automatically
> discovered by sops. You'll need to pass the `--config .sops.yml` option for it to be picked up.

Let\'s take an example:

-   file named **something.dev.yaml** should use one set of KMS A, PGP
    and age
-   file named **something.prod.yaml** should use another set of KMS B,
    PGP and age
-   other files use a third set of KMS C and PGP
-   all live under **mysecretrepo/something.{dev,prod,gcp}.yaml**

Under those circumstances, a file placed at **mysecretrepo/.sops.yaml**
can manage the three sets of configurations for the three types of
files:

``` yaml
# creation rules are evaluated sequentially, the first match wins
creation_rules:
    # upon creation of a file that matches the pattern *.dev.yaml,
    # KMS set A as well as PGP and age is used
    - path_regex: \.dev\.yaml$
      kms: 'arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500,arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e+arn:aws:iam::361527076523:role/hiera-sops-prod'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'
      age: 'age129h70qwx39k7h5x6l9hg566nwm53527zvamre8vep9e3plsm44uqgy8gla'

    # prod files use KMS set B in the PROD IAM, PGP and age
    - path_regex: \.prod\.yaml$
      kms: 'arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e+arn:aws:iam::361527076523:role/hiera-sops-prod,arn:aws:kms:eu-central-1:361527076523:key/cb1fab90-8d17-42a1-a9d8-334968904f94+arn:aws:iam::361527076523:role/hiera-sops-prod'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'
      age: 'age129h70qwx39k7h5x6l9hg566nwm53527zvamre8vep9e3plsm44uqgy8gla'
      hc_vault_uris: "http://localhost:8200/v1/sops/keys/thirdkey"

    # gcp files using GCP KMS
    - path_regex: \.gcp\.yaml$
      gcp_kms: projects/mygcproject/locations/global/keyRings/mykeyring/cryptoKeys/thekey

    # hckms files using HuaweiCloud KMS
    - path_regex: \.hckms\.yaml$
      hckms: tr-west-1:abc12345-6789-0123-4567-890123456789,tr-west-2:def67890-1234-5678-9012-345678901234

    # Finally, if the rules above have not matched, this one is a
    # catchall that will encrypt the file using KMS set C as well as PGP
    # The absence of a path_regex means it will match everything
    - kms: 'arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500,arn:aws:kms:us-west-2:142069644989:key/846cfb17-373d-49b9-8baf-f36b04512e47,arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'
```

When creating any file under **mysecretrepo**, whether at the root or
under a subdirectory, SOPS will recursively look for a `.sops.yaml`
file. If one is found, the filename of the file being created is
compared with the filename regexes of the configuration file. The first
regex that matches is selected, and its KMS and PGP keys are used to
encrypt the file. It should be noted that the looking up of `.sops.yaml`
is from the working directory (CWD) instead of the directory of the
encrypting file (see [Issue
242](https://github.com/getsops/sops/issues/242)).

The `path_regex` checks the path of the encrypting file relative to the
`.sops.yaml` config file. Here is another example:

-   files located under directory **development** should use one set of
    KMS A
-   files located under directory **production** should use another set
    of KMS B
-   other files use a third set of KMS C

``` yaml
creation_rules:
    # upon creation of a file under development,
    # KMS set A is used
    - path_regex: .*/development/.*
      kms: 'arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500,arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e+arn:aws:iam::361527076523:role/hiera-sops-prod'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'

    # prod files use KMS set B in the PROD IAM
    - path_regex: .*/production/.*
      kms: 'arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e+arn:aws:iam::361527076523:role/hiera-sops-prod,arn:aws:kms:eu-central-1:361527076523:key/cb1fab90-8d17-42a1-a9d8-334968904f94+arn:aws:iam::361527076523:role/hiera-sops-prod'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'

    # other files use KMS set C
    - kms: 'arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500,arn:aws:kms:us-west-2:142069644989:key/846cfb17-373d-49b9-8baf-f36b04512e47,arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e'
      pgp: 'FBC7B9E2A4F9289AC0C1D4843D16CEE4A27381B4'
```

Creating a new file with the right keys is now as simple as

``` sh
$ sops edit <newfile>.prod.yaml
```

Note that the configuration file is ignored when KMS or PGP parameters
are passed on the SOPS command line or in environment variables.
