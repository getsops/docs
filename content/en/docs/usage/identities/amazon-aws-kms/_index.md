---
title: "Amazon AWS KMS"
weight: 30
---

## KMS AWS Profiles

If you want to use a specific profile, you can do so with
\`aws_profile\`:

``` yaml
sops:
    kms:
        - arn: arn:aws:kms:us-east-1:656532927350:key/920aff2e-c5f1-4040-943a-047fa387b27e
          aws_profile: foo
```

If no AWS profile is set, default credentials will be used.

Similarly the `--aws-profile` flag can be set with the
command line with any of the KMS commands.

## Assuming roles and using KMS in various AWS accounts

SOPS has the ability to use KMS in multiple AWS accounts by assuming
roles in each account. Being able to assume roles is a nice feature of
AWS that allows administrators to establish trust relationships between
accounts, typically from the most secure account to the least secure
one. In our use-case, we use roles to indicate that a user of the Master
AWS account is allowed to make use of KMS master keys in development and
staging AWS accounts. Using roles, a single file can be encrypted with
KMS keys in multiple accounts, thus increasing reliability and ease of
use.

You can use keys in various accounts by tying each KMS master key to a
role that the user is allowed to assume in each account. The [IAM
roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html)
documentation has full details on how this needs to be configured on
AWS\'s side.

From the point of view of SOPS, you only need to specify the role a KMS
key must assume alongside its ARN, as follows:

``` yaml
sops:
    kms:
        - arn: arn:aws:kms:us-east-1:656532927350:key/920aff2e-c5f1-4040-943a-047fa387b27e
          role: arn:aws:iam::927034868273:role/sops-dev-xyz
```

The role must have permission to call Encrypt and Decrypt using KMS. An
example policy is shown below.

``` json
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*",
  "Principal": {
    "AWS": [
      "arn:aws:iam::927034868273:role/sops-dev-xyz"
    ]
  }
}
```

You can specify a role in the `--kms` flag and `SOPS_KMS_ARN` variable
by appending it to the ARN of the master key, separated by a **+** sign:

```
<KMS ARN>+<ROLE ARN>
arn:aws:kms:us-west-2:927034868273:key/fe86dd69-4132-404c-ab86-4269956b4500+arn:aws:iam::927034868273:role/sops-dev-xyz
```

## AWS KMS Encryption Context

SOPS has the ability to use [AWS KMS key policy and encryption
context](http://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html)
to refine the access control of a given KMS master key.

When creating a new file, you can specify the encryption context in the
`--encryption-context` flag by comma separated list of key-value pairs:

``` sh
$ sops edit --encryption-context Environment:production,Role:web-server test.dev.yaml
```

The format of the Encrypt Context string is
`<EncryptionContext Key>:<EncryptionContext Value>,<EncryptionContext Key>:<EncryptionContext Value>,...`

The encryption context will be stored in the file metadata and does not
need to be provided at decryption.

Encryption contexts can be used in conjunction with KMS Key Policies to
define roles that can only access a given context. An example policy is
shown below:

``` json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/RoleForExampleApp"
  },
  "Action": "kms:Decrypt",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:AppName": "ExampleApp",
      "kms:EncryptionContext:FilePath": "/var/opt/secrets/"
    }
  }
}
```
