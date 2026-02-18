---
title: "SOPS: Secrets OPerationS"
description: Simple And Flexible Tool For Managing Secrets  
---

{{% blocks/cover color="white" title="SOPS: Secrets OPerationS" height="full td-below-navbar" %}}
  {{% _param description %}}
  {.display-6}

  <div class="flex flex-column align-items-center">
  <p class="hero-text-secondary">
    <span class="hero-tags">Encrypt configuration</span>
    <span class="hero-tags">Keep structure visible</span>
    <span class="hero-tags">Access management through identities</span>
    <span class="hero-tags">Offline and cloud based identities</span>
  </p>
  </div>

  <div class="td-cta-buttons my-5">
    <a {{% _param btn-lg primary %}} href="docs/">
      Learn more
    </a>
    <a {{% _param btn-lg secondary %}} href="/docs/#download">
      Download
    </a>
  </div>

  {{% blocks/link-down %}}
{{% /blocks/cover %}}

{{% blocks/section color="white" type="row" %}}

  {{% blocks/feature title="Encrypt configuration sensibly" icon="fa-exchange fa-lg" %}}
  SOPS encrypts configuration files while keeping the structure visible.
  Keys are not encrypted, while values and comments are encrypted.
  This allows you to understand the configuration without seeing sensible values.
  Also commented-out secrets aren't suddenly visible to everyone!
  {{% /blocks/feature %}}

  {{% blocks/feature title="Various config file formats" icon="fa-box-open fa-lg" %}}
  SOPS supports [YAML](https://yaml.org/), [JSON](https://www.json.org/),
  and specific flavors of [INI](https://en.wikipedia.org/wiki/INI_file) and DotEnv configuration files.
  You can also encrypt files completely through SOPS' "binary" store.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Managing access through identities" icon="fa-users fa-lg" %}}
  Access to configuration is managed through identities.
  You can configure a set of identities that can access a file,
  and also require multiple identities together that a user needs access to to decrypt a file.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Works offline and online" icon="fa-plug fa-lg" %}}
  SOPS can use offline methods (Age, PGP/GnuPG)
  and online methods (cloud based KMSes, secret management software)
  to encrypt and decrypt a configuration's session key.
  You can use SOPS in cloud infrastructure and also locally for disaster recovery.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Security" icon="fa-key fa-lg" %}}
  The security of the data stored using SOPS is as strong as the weakest
  cryptographic mechanism.
  Values are encrypted using [AES256](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
  in [GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode) mode.
  How secure the key is stored depends on the identities used.
  For example, you can use hybrid [post-quantum cryptographic](https://en.wikipedia.org/wiki/Post-quantum_cryptography) encryption through [Age](https://age-encryption.org/).
  {{% /blocks/feature %}}

  {{% blocks/feature title="Key stores" icon="fa-vector-square fa-lg" %}}
  SOPS supports [Age](https://age-encryption.org/) and PGP/[GnuPG](https://www.gnupg.org/) for offline identities,
  and [Amazon AWS KMS](https://aws.amazon.com/kms/), [Google Cloud KMS](https://docs.cloud.google.com/kms/docs),
  [Azure KMS](https://en.wikipedia.org/wiki/Microsoft_Azure), [HuaweiCloud KMS](https://cloud.huawei.com/),
  [HashiCorp Vault](https://www.hashicorp.com/en/products/vault), and [OpenBAO](https://openbao.org/) for online identities.
  {{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="white" type="row text-center h1" %}}

  SOPS is a [Cloud Native Computing Foundation sandbox project](https://www.cncf.io/sandbox-projects/)

  <div class="img-container">
  <img style="max-width: 400px;" class="dark-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-white.svg">
  <img style="max-width: 400px;" class="light-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-black.svg">
  </div>

{{% /blocks/section %}}
