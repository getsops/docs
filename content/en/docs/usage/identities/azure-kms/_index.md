---
title: "Azure KMS"
weight: 40
description: You can use Azure's KMS to encrypt data.
---

## Encrypting using Azure Key Vault

The Azure Key Vault integration uses the [default credential
chain](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#DefaultAzureCredential)
which tries several authentication methods, in this order:

1.  [Environment
    credentials](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#EnvironmentCredential)

    1. Service Principal with Client Secret
    2. Service Principal with Certificate
    3. User with username and password
    4. Configuration for multi-tenant applications

2.  [Workload Identity
    credentials](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#WorkloadIdentityCredential)
3.  [Managed Identity
    credentials](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#ManagedIdentityCredential)
4.  [Azure CLI
    credentials](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#AzureCLICredential)

For example, you can use a Service Principal with the following
environment variables:

``` bash
AZURE_TENANT_ID
AZURE_CLIENT_ID
AZURE_CLIENT_SECRET
```

You can create a Service Principal using the CLI like this:

``` sh
$ az ad sp create-for-rbac -n my-keyvault-sp

{
    "appId": "<some-uuid>",
    "displayName": "my-keyvault-sp",
    "name": "http://my-keyvault-sp",
    "password": "<random-string>",
    "tenant": "<tenant-uuid>"
}
```

The `appId` is the client ID, and the `password`
is the client secret.

Encrypting/decrypting with Azure Key Vault requires the resource
identifier for a key. This has the following form:

```
https://${VAULT_URL}/keys/${KEY_NAME}/${KEY_VERSION}
```

You can omit the version, and have just a trailing slash, and this will use
whatever the latest version of the key is:

```
https://${VAULT_URL}/keys/${KEY_NAME}/
```

To create a Key Vault and assign your service principal permissions on
it from the commandline:

``` sh
# Create a resource group if you do not have one:
$ az group create --name sops-rg --location westeurope
# Key Vault names are globally unique, so generate one:
$ keyvault_name=sops-$(uuidgen | tr -d - | head -c 16)
# Create a Vault, a key, and give the service principal access:
$ az keyvault create --name $keyvault_name --resource-group sops-rg --location westeurope
$ az keyvault key create --name sops-key --vault-name $keyvault_name --protection software --ops encrypt decrypt
$ az keyvault set-policy --name $keyvault_name --resource-group sops-rg --spn $AZURE_CLIENT_ID \
    --key-permissions get encrypt decrypt
# Read the key id:
$ az keyvault key show --name sops-key --vault-name $keyvault_name --query key.kid

https://sops.vault.azure.net/keys/sops-key/some-string
```

> 📝 **Note**
>
> The `get` key permission is required when the key version is ommited (for example if the URL ends with a trailing slash).
> In that case SOPS calls the Azure Key Vault API to resolve the latest key version, which requires the `get` permission.
> If you specifty an explicit key version in the URL you can omit `get`, but this means you will need to update your configuration every time the key is rotated.

Now you can encrypt a file using:

``` sh
$ sops encrypt --azure-kv https://sops.vault.azure.net/keys/sops-key/some-string test.yaml > test.enc.yaml
```

or, without the version:

``` sh
$ sops encrypt --azure-kv https://sops.vault.azure.net/keys/sops-key/ test.yaml > test.enc.yaml
```

And decrypt it using:

``` sh
$ sops decrypt test.enc.yaml
```
