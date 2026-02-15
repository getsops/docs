---
title: "SOPS: Secrets OPerationS"
description: Simple And Flexible Tool For Managing Secrets  
---

{{% blocks/cover color="white" title="SOPS: Secrets OPerationS" height="full td-below-navbar" %}}
  {{% _param description %}}
  {.display-6}

  <div class="flex flex-column align-items-center">
  <p class="hero-text-secondary">
    <span class="hero-tags">Decrypt a file</span>
    <span class="hero-tags">Encrypt a file using AWS</span>
    <span class="hero-tags">Encrypt a file using GCP</span>
    <span class="hero-tags">Encrypt a file using Azure</span>
    <span class="hero-tags">Encrypt a file using Age</span>
    <span class="hero-tags">Encrypt a file using PGP</span>
    <span class="hero-tags">Edit encrypted file</span>
    <span class="hero-tags">Edit decrypted file</span>
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

  {{% blocks/feature title="Key Groups" icon="fab fa-exchange fa-lg" %}}
  By default, SOPS encrypts the data key for a file with each of the
  master keys, such that if any of the master keys is available, the
  file can be decrypted. However, it is sometimes desirable to require
  access to multiple master keys in order to decrypt files. This can be
  achieved with key groups.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Auditing" icon="fab fa-box-open fa-lg" %}}
  Sometimes, users want to be able to tell what files were accessed by
  whom in an environment they control. For this reason, SOPS can
  generate audit logs to record activity on encrypted files. When
  enabled, SOPS will write a log entry into a pre-configured PostgreSQL
  database when a file is decrypted.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Key Service" icon="fab fa-plug fa-lg" %}}
  There are situations where you might want to run SOPS on a machine
  that doesn't have direct access to encryption keys such as PGP keys.
  The sops key service allows you to forward a socket so that SOPS can
  access encryption keys stored on a remote machine.
  {{% /blocks/feature %}}

  {{% blocks/feature title="Security" icon="fab fa-vector-square fa-lg" %}}
  The security of the data stored using SOPS is as strong as the weakest
  cryptographic mechanism. Values are encrypted using AES256_GCM which
  is the strongest symmetric encryption algorithm known today. Data keys
  are encrypted in either KMS, which also uses AES256_GCM, or PGP which
  uses either RSA or ECDSA keys.
  {{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="white" type="row text-center h1" %}}

  SOPS is a [Cloud Native Computing Foundation sandbox project](https://www.cncf.io/sandbox-projects/)

  <div class="img-container">
  <img style="max-width: 400px;" class="dark-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-white.svg">
  <img style="max-width: 400px;" class="light-mode-only" alt="CNCF Sandbox Project" src="/images/cncf-sandbox-horizontal-black.svg">
  </div>

{{% /blocks/section %}}
