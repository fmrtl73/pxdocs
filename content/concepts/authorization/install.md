---
title: PX-Security on an existing cluster
description: Explains how to enable PX-Security in Portworx on an exiting cluster 
keywords: enable authorization, security, RBAC, Role Based Access Control, JWT, JSON Web Token, self-signed, claims, upgrade to auth, auth enabled cluster
weight: 3000
series: authorization
---

This page guides you to enable PX-Security on an existing cluster with no PX-Security setup.

## Secure your existing cluster

Perform the following steps to enable PX-Security:

1. Enable PX-Security in Portworx on each node. You can refer to the following sections depending on your deployment:

 * **Operator deployment:** [Secure your storage with the Operator](/cloud-references/security/kubernetes/shared-secret-model-operator/)


 * **DaemonSet deployment:** [Secure your storage with a DaemonSet](/cloud-references/security/kubernetes/shared-secret-model/)

    {{<info>}}
   **Note:**
    To completely secure the cluster, you must enable security on all nodes participating in the Portworx cluster. Do not mix the nodes with enabled and disabled PX-Security.
   {{</info>}}

2. Generate a new cluster token.
Run the following command to generate a new cluster token for pairing and migrating your clusters:

```text
pxctl cluster token reset
```

## Security parameters

The following parameters are required and must be provided to Portworx to enable PX-Security.

### Environment variables 
Provide sensitive information like shared secrets as environment variables. These variables can be provided by _secrets_ through your container orchestration system.

| Environment Variable | Required? | Description |
| -------------------- | --------- | ----------- |
| `PORTWORX_AUTH_SYSTEM_KEY` | Yes | Shared secret used by Portworx to generate tokens for cluster communications |
| `PORTWORX_AUTH_SYSTEM_APPS_KEY` | Yes when using Stork | Share secret used by Stork to generate tokens to communicate with Portworx. The shared secret must match the value of `PX_SHARED_SECRET` environment variable in Stork. |
| `PORTWORX_AUTH_JWT_SHAREDSECRET` | Optional | Self-generated token shared secret, if any |

### Configuration 

For non-sensitive information, use the`px-runc`command as command-line parameters with the following arguments:

| Name | Description |
| ---- | ----------- |
| `-jwt_issuer <issuer>` | JSON Web Token issuer (e.g. openstorage.io). This is the token issuer for your self-signed tokens. It must match the `iss` value in token claims |
| `-jwt_rsa_pubkey_file <file path>` | JSON Web Token RSA Public file path |
| `-jwt_ecds_pubkey_file <file path>` | JSON Web Token ECDS Public file path |
| `-username_claim <claim>` | Name of the claim in the token to be used as the unique ID of the user (<claim> can be `sub`, `email` or `name`, default: `sub`) |