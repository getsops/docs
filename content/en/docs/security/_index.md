---
title: "Security"
weight: 60
---

## Reporting security issues

Please report any security issues privately using [GitHub\'s advisory
form](https://github.com/getsops/sops/security/advisories).

## Motivation

> 📝 **A note from the maintainers**
>
> This section was written by the original authors of SOPS while they
> were working at Mozilla. It is kept here for historical reasons and to
> provide technical background on the project. It is not necessarily
> representative of the views of the current maintainers, nor are they
> currently affiliated with Mozilla.

Automating the distribution of secrets and credentials to components of
an infrastructure is a hard problem. We know how to encrypt secrets and
share them between humans, but extending that trust to systems is
difficult. Particularly when these systems follow devops principles and
are created and destroyed without human intervention. The issue boils
down to establishing the initial trust of a system that just joined the
infrastructure, and providing it access to the secrets it needs to
configure itself.

### The initial trust

In many infrastructures, even highly dynamic ones, the initial trust is
established by a human. An example is seen in Puppet by the way
certificates are issued: when a new system attempts to join a
Puppetmaster, an administrator must, by default, manually approve the
issuance of the certificate the system needs. This is cumbersome, and
many puppetmasters are configured to auto-sign new certificates to work
around that issue. This is obviously not recommended and far from ideal.

AWS provides a more flexible approach to trusting new systems. It uses a
powerful mechanism of roles and identities. In AWS, it is possible to
verify that a new system has been granted a specific role at creation,
and it is possible to map that role to specific resources. Instead of
trusting new systems directly, the administrator trusts the AWS
permission model and its automation infrastructure. As long as AWS keys
are safe, and the AWS API is secure, we can assume that trust is
maintained and systems are who they say they are.

### KMS, Trust and secrets distribution

Using the AWS trust model, we can create fine grained access controls to
Amazon\'s Key Management Service (KMS). KMS is a service that encrypts
and decrypts data with AES_GCM, using keys that are never visible to
users of the service. Each KMS master key has a set of role-based access
controls, and individual roles are permitted to encrypt or decrypt using
the master key. KMS helps solve the problem of distributing keys, by
shifting it into an access control problem that can be solved using
AWS\'s trust model.

### Operational requirements

When Mozilla\'s Services Operations team started revisiting the issue of
distributing secrets to EC2 instances, we set a goal to store these
secrets encrypted until the very last moment, when they need to be
decrypted on target systems. Not unlike many other organizations that
operate sufficiently complex automation, we found this to be a hard
problem with a number of prerequisites:

1.  Secrets must be stored in YAML files for easy integration into hiera
2.  Secrets must be stored in GIT, and when a new CloudFormation stack
    is built, the current HEAD is pinned to the stack. (This allows
    secrets to be changed in GIT without impacting the current stack
    that may autoscale).
3.  Entries must be encrypted separately. Encrypting entire files as
    blobs makes git conflict resolution almost impossible. Encrypting
    each entry separately is much easier to manage.
4.  Secrets must always be encrypted on disk (admin laptop, upstream git
    repo, jenkins and S3) and only be decrypted on the target systems

SOPS can be used to encrypt YAML, JSON, ENV, INI, and BINARY files. In BINARY mode,
the content of the file is treated as a blob, the same way PGP would
encrypt an entire file. In YAML, JSON, ENV, and INI modes, however, the content of
the file is manipulated as a tree where keys are stored in cleartext,
and values are encrypted. hiera-eyaml does something similar, and over
the years we learned to appreciate its benefits, namely:

-   diffs are meaningful. If a single value of a file is modified, only
    that value will show up in the diff. The diff is still limited to
    only showing encrypted data, but that information is already more
    granular that indicating that an entire file has changed.
-   conflicts are easier to resolve. If multiple users are working on
    the same encrypted files, as long as they don\'t modify the same
    values, changes are easy to merge. This is an improvement over the
    PGP encryption approach where unsolvable conflicts often happen when
    multiple users work on the same file.

### OpenPGP integration

OpenPGP gets a lot of bad press for being an outdated crypto protocol,
and while true, what really made us look for alternatives is the
difficulty of managing and distributing keys to systems. With KMS, we
manage permissions to an API, not keys, and that\'s a lot easier to do.

But PGP is not dead yet, and we still rely on it heavily as a backup
solution: all our files are encrypted with KMS and with one PGP public
key, with its private key stored securely for emergency decryption in
the event that we lose all our KMS master keys.

SOPS can be used without KMS entirely, the same way you would use an
encrypted PGP file: by referencing the pubkeys of each individual who
has access to the file. It can easily be done by providing SOPS with a
comma-separated list of public keys when creating a new file:

``` sh
$ sops edit --pgp "E60892BB9BD89A69F759A1A0A3D652173B763E8F,84050F1D61AF7C230A12217687DF65059EF093D3,85D77543B3D624B63CEA9E6DBC17301B491B3F21" mynewfile.yaml
```

## Threat Model

The security of the data stored using SOPS is as strong as the weakest
cryptographic mechanism. Values are encrypted using AES256_GCM which is
the strongest symmetric encryption algorithm known today. Data keys are
encrypted in either KMS, which also uses AES256_GCM, or PGP which uses
either RSA or ECDSA keys.

Going from the most likely to the least likely, the threats are as
follows:

### Compromised AWS credentials grant access to KMS master key

An attacker with access to an AWS console can grant itself access to one
of the KMS master keys used to encrypt a `sops` data key. This threat
should be mitigated by protecting AWS accesses with strong controls,
such as multi-factor authentication, and also by performing regular
audits of permissions granted to AWS users.

### Compromised PGP key

PGP keys are routinely mishandled, either because owners copy them from
machine to machine, or because the key is left forgotten on an unused
machine an attacker gains access to. When using PGP encryption, SOPS
users should take special care of PGP private keys, and store them on
smart cards or offline as often as possible.

### Factorized RSA key

SOPS doesn\'t apply any restriction on the size or type of PGP keys. A
weak PGP keys, for example 512 bits RSA, could be factorized by an
attacker to gain access to the private key and decrypt the data key.
Users of SOPS should rely on strong keys, such as 2048+ bits RSA keys,
or 256+ bits ECDSA keys.

### Weak AES cryptography

A vulnerability in AES256_GCM could potentially leak the data key or the
KMS master key used by a SOPS encrypted file. While no such
vulnerability exists today, we recommend that users keep their encrypted
files reasonably private.
