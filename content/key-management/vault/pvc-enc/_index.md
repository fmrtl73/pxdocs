---
title: Encrypting Kubernetes PVCs with Vault
weight: 100
keywords: Vault Key Management, Hashicorp, encrypt PVC, Kubernetes, k8s, Vault Namespaces
description: Instructions on using Vault with Portworx for encrypting PVCs in Kubernetes
noicon: true
series: vault-secret-uses
series2: k8s-pvc-enc
hidden: true
---

{{< content "shared/key-management-intro.md" >}}

There are two ways in which Portworx volumes can be encrypted and are dependent on how the Vault secret key is provided to Portworx.

### Encryption using Storage Class

In this method, Portworx uses the cluster wide secret key to encrypt PVCs.

  ![Concept](/img/vault-px-concept.png)

- By default, Portworx looks for the secret in path `secret/<secret-name>` and gets the first value under the path. Users can modify the path with BACKEND_PATH or BASE_PATH, depending on the vault's version.
- The **cluster wide secret** is the global secret-name for the cluster.
- secret-name can be modified with annotations. See the next section for an example.

From the diagram:

- The top-most secret can be accessed with default BACKEND_PATH `secret/` and secret-name `px-vault`.
- The middle secret can be accessed with default BACKEND_PATH `secret/` and secret-name `custom/name`.
- The bottom secret can be accessed with BACKEND_PATH `aws-secret/` and secret-name `aws-vault`.


#### Step 1: Store the secret in Vault


```text
vault kv put secret/px-vault foo=bar 
```

#### Step 2: Set a cluster wide secret

{{< content "shared/key-management-set-cluster-wide-secret.md" >}}

{{<info>}}
**NOTE:** To use the secret we just created, enter `px-vault` for the the cluster wide sceret.
{{</info>}}

If you are using Vault Namespaces use the following command to set the cluster-wide secret key in a specific vault namespace:

```text
pxctl secrets set-cluster-key --secret_options=vault-namespace=<name of vault-namespace>
```

{{< content "shared/key-management-storage-class-encryption.md" >}}

### Encryption using PVC annotations

In this method, each PVC can be encrypted with its own secret key.

#### Step 1: Create a Storage Class

{{< content "shared/key-management-enc-storage-class-spec.md" >}}

#### Step 2: Create a PVC with annotations

{{< content "shared/key-management-other-providers-pvc-encryption.md" >}}

{{<info>}}
**IMPORTANT:** Make sure secret `your-secret-name` exists in Vault.
{{</info>}}

### Encryption using PVC annotations with Vault Namespaces

If you have **Vault Namespaces** enabled and your secret resides inside a specific namespace, you must provide the name of that namespace and the secret key to Portworx.

#### Step 1: Create a Storage Class

{{< content "shared/key-management-enc-storage-class-spec.md" >}}

#### Step 2: Create a PVC with annotations

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
  annotations:
    px/secret-name: <your-secret-name>
    px/vault-namespace: <your-vault-namesapce>
spec:
  storageClassName: px-secure-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

The PVC requires an extra annotation `px/vault-namespace` to indicate the Vault namespace where the secret key resides. If your key resides in the global vault namespace
set in Portworx using the parameter `VAULT_NAMESPACE`, you don't need to specify this annotation. However if the key resides in any other namespace then this annotation is
required.

{{<info>}}
**IMPORTANT:** Make sure the secret `your-secret-key` exists in the namespace `your-vault-namespace` in Vault.
{{</info>}}
