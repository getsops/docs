---
title: "SOPS: Secrets OPerationS"
linkTitle: Docs
menu: {main: {weight: 20}}
weight: 20
---

## Introduction

**SOPS** is an editor of encrypted files that supports YAML, JSON, ENV,
INI and BINARY formats and encrypts with AWS KMS, GCP KMS, Azure Key
Vault, HuaweiCloud KMS, age, and PGP.
([demo](https://www.youtube.com/watch?v=YTEVyLXFiq0))

![image](https://i.imgur.com/X0TM5NI.gif)

------------------------------------------------------------------------

[![image](https://pkg.go.dev/badge/github.com/getsops/sops/v3.svg)](https://pkg.go.dev/github.com/getsops/sops/v3)

## More information on...

- [Installation](installation/)
- [Usage](usage/)
- [Reference](reference/)
- [Security](security/)

## Backward compatibility

SOPS will remain backward compatible on the major version, meaning that
all improvements brought to the 1.X and 2.X branches (current) will
maintain the file format introduced in **1.0**.

## License

Mozilla Public License Version 2.0

## Authors

SOPS was initially launched as a project at Mozilla in 2015 and has been
graciously donated to the CNCF as a Sandbox project in 2023, now under
the stewardship of a [new group of
maintainers](https://github.com/getsops/community/blob/main/MAINTAINERS.md).

The original authors of the project were:

-   Adrian Utrilla \@autrilla
-   Julien Vehent \@jvehent

Furthermore, the project has been carried for a long time by AJ Bahnken
\@ajvb, and had not been possible without the contributions of numerous
[contributors](https://github.com/getsops/sops/graphs/contributors).

## Credits

SOPS was inspired by
[hiera-eyaml](https://github.com/TomPoulton/hiera-eyaml),
[credstash](https://github.com/LuminalOSS/credstash),
[sneaker](https://github.com/codahale/sneaker), [password
store](http://www.passwordstore.org/) and too many years managing PGP
encrypted files by hand\...

------------------------------------------------------------------------

<img style="max-width: 400px;" class="dark-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-white.svg">
<img style="max-width: 400px;" class="light-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-black.svg">

**We are a [Cloud Native Computing Foundation](https://cncf.io) sandbox project.**
