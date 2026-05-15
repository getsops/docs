---
title: "Google Cloud KMS"
weight: 50
description: You can use Google Cloud's KMS to encrypt data.
---

## Encrypting using GCP KMS

GCP KMS has support for authorization with the use of
[Application Default Credentials](https://developers.google.com/identity/protocols/application-default-credentials)
and using an OAuth 2.0 token. Application default credentials precedes
the use of access token.

Using Application Default Credentials you can authorize by doing this:

``` sh
$ gcloud auth login
```

you can enable application default credentials using the sdk:

``` sh
$ gcloud auth application-default login
```

Using OAauth tokens you can authorize by doing this:

``` sh
$ export GOOGLE_OAUTH_ACCESS_TOKEN=<your access token>
```

Or if you are logged in you can authorize by generating an access token:

``` sh
$ export GOOGLE_OAUTH_ACCESS_TOKEN="$(gcloud auth print-access-token)"
```

By default, SOPS uses the gRPC client to communicate with GCP KMS. You can optionally
switch to the REST client by setting the `SOPS_GCP_KMS_CLIENT_TYPE` environment variable:

``` sh
$ export SOPS_GCP_KMS_CLIENT_TYPE=rest  # Use REST client
$ export SOPS_GCP_KMS_CLIENT_TYPE=grpc  # Use gRPC client (default)
```

For sovereign cloud environments that expose a GCP-compatible KMS API at a
non-standard endpoint (e.g. S3NS/Thales TPC: `cloudkms.s3nsapis.fr`),
you can override the endpoint or the universe domain:

``` sh
# Override the KMS endpoint directly
$ export SOPS_GCP_KMS_ENDPOINT=cloudkms.example.com:443

# Or derive the endpoint from the universe domain (cloudkms.<domain>:443)
$ export SOPS_GCP_KMS_UNIVERSE_DOMAIN=example.com
```

> 📝 **Note**
>
> `SOPS_GCP_KMS_ENDPOINT` takes precedence over `SOPS_GCP_KMS_UNIVERSE_DOMAIN` if both are set.

Encrypting/decrypting with GCP KMS requires a KMS ResourceID. You can
use the cloud console the get the ResourceID or you can create one using
the gcloud sdk:

``` sh
$ gcloud kms keyrings create sops --location global
$ gcloud kms keys create sops-key --location global --keyring sops --purpose encryption
$ gcloud kms keys list --location global --keyring sops

# you should see
NAME                                                                   PURPOSE          PRIMARY_STATE
projects/my-project/locations/global/keyRings/sops/cryptoKeys/sops-key ENCRYPT_DECRYPT  ENABLED
```

Now you can encrypt a file using:

``` sh
$ sops encrypt --gcp-kms projects/my-project/locations/global/keyRings/sops/cryptoKeys/sops-key test.yaml > test.enc.yaml
```

And decrypt it using:

``` sh
$ sops decrypt test.enc.yaml
```
