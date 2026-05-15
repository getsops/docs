---
title: "HashiCorp Vault / OpenBao"
weight: 60
description: You can use HashiCorp Vault or OpenBao to encrypt data.
---

## Encrypting using Hashicorp Vault

We assume you have an instance (or more) of Vault running and you have
privileged access to it. For instructions on how to deploy a secure
instance of Vault, refer to Hashicorp\'s official documentation.

To easily deploy Vault locally: (DO NOT DO THIS FOR PRODUCTION!!!)

``` sh
$ docker run -d -p8200:8200 vault:1.2.0 server -dev -dev-root-token-id=toor
```

``` sh
$ # Substitute this with the address Vault is running on
$ export VAULT_ADDR=http://127.0.0.1:8200 

$ # this may not be necessary in case you previously used `vault login` for production use
$ export VAULT_TOKEN=toor 

$ # to check if Vault started and is configured correctly
$ vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.2.0
Cluster Name    vault-cluster-618cc902
Cluster ID      e532e461-e8f0-1352-8a41-fc7c11096908
HA Enabled      false

$ # It is required to enable a transit engine if not already done (It is suggested to create a transit engine specifically for SOPS, in which it is possible to have multiple keys with various permission levels)
$ vault secrets enable -path=sops transit
Success! Enabled the transit secrets engine at: sops/

$ # Then create one or more keys
$ vault write sops/keys/firstkey type=rsa-4096
Success! Data written to: sops/keys/firstkey

$ vault write sops/keys/secondkey type=rsa-2048
Success! Data written to: sops/keys/secondkey

$ vault write sops/keys/thirdkey type=chacha20-poly1305
Success! Data written to: sops/keys/thirdkey

$ sops encrypt --hc-vault-transit $VAULT_ADDR/v1/sops/keys/firstkey vault_example.yml

$ cat <<EOF > .sops.yaml
creation_rules:
    - path_regex: \.dev\.yaml$
      hc_vault_transit_uri: "$VAULT_ADDR/v1/sops/keys/secondkey"
    - path_regex: \.prod\.yaml$
      hc_vault_transit_uri: "$VAULT_ADDR/v1/sops/keys/thirdkey"
EOF

$ sops encrypt --verbose prod/raw.yaml > prod/encrypted.yaml
```

### Restricting HC Vault servers that SOPS can talk to

If you want to restrict which HC Vault servers SOPS is allowed to talk to, you can set the `SOPS_HC_VAULT_ALLOWLIST` environment variable.
When set to `all` (the default value), there is no restriction.
When set to `none`, SOPS will not allow any access to HC Vault servers for decryption or encryption.

When set to any other value, this value will be interpreted as a comma-separated list of strings.
If SOPS attempts to contact a vault URL that starts with one of these strings, SOPS will attempt to contact that URL.
If there is no matching prefix in `SOPS_HC_VAULT_ALLOWLIST`, SOPS will not contact that URL.
