---
title: Encrypting PVCs using CSI and Kubernetes Secrets
linkTitle: PVC Encryption with CSI and Kubernetes
weight: 100
keywords: Portworx, Kubernetes, Kubernetes Secrets, containers, storage, encryption, CSI
description: Instructions on using Kubernetes Secrets with Portworx for encrypting PVCs on CSI using StorageClass
noicon: true
series: kubernetes-secret-uses
series2: k8s-pvc-enc
hidden: false
aliases:
    - /key-management/kubernetes-secrets/pvc-encryption-using-CSI/
---
This article discusses the PVC encryption methods used with the Kubernetes Container Storage Interface. For details about using Portworx with CSI, refer to the [Portworx with CSI](/operations/operate-kubernetes/storage-operations/csi/) page.

## Prerequisites

In order to perform the steps in this document, you must have [Portworx with CSI](/operations/operate-kubernetes/storage-operations/csi/) version 1 enabled.

## Encrypt your volumes

You can encrypt your volumes in one of two ways:

* Per storage class
* Per PVC

### Encrypt your volumes per storage class

You can encrypt your volumes by specifying the encryption key in a Kubernetes secret. This secret can be same as the one created to host the authentication token. Using this method, you can handle both authentication and encryption together, and multiple PVCs referring to this storage class can use the same secret for encryption.

#### Step 1: Create a Kubernetes secret that contains the passphrase used for encrypting the Portworx volume

Enter the following `kubectl create secret generic` command, specifying your own passphrase in `mysecret-passcode-for-encryption`, which encrypts the PVC:

```text
kubectl create secret generic volume-secrets -n kube-system --from-literal=mysql-pvc-secret-key=mysecret-passcode-for-encryption
```

#### Step 2: Create a CSI Kubernetes secret that points to the Kubernetes encryption secret

The CSI implementation reads the Kubernetes secret `px-secret` and passes its contents to Portworx. The `px-secret` must contain the `SECRET_NAME`, `SECRET_CONTEXT`, and `SECRET_KEY`. That is how Portworx knows which Kubernetes secret to fetch the encryption passphrase from.

Enter the following `kubectl create secret generic` command:

```text
kubectl create secret generic px-secret -n kube-system --from-literal=SECRET_NAME=volume-secrets --from-literal=SECRET_KEY=mysql-pvc-secret-key --from-literal=SECRET_CONTEXT=portworx
```

#### Step 3: Create a CSI storage class for encrypted PVC

Create the storage class which refers to the CSI secret you created in step 2 above. Specify the following:

  * `csi.storage.k8s.io/provisioner-secret-name` and the name of your CSI secret
  * `csi.storage.k8s.io/provisioner-secret-namespace` and the namespace in which your CSI secret is located
  * `csi.storage.k8s.io/node-publish-secret-name` and the name of your CSI secret
  * `csi.storage.k8s.io/node-publish-secret-namespace` and the namespace in which your CSI secret is located

    ```text
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: px-csi-db-encrypted-k8s
    provisioner: pxd.portworx.com
    parameters:
      repl: "3"
      secure: "true"
      io_profile: auto
      io_priority: "high" 
      cow_ondemand: "false"
      csi.storage.k8s.io/provisioner-secret-name: px-secret
      csi.storage.k8s.io/provisioner-secret-namespace: kube-system
      csi.storage.k8s.io/node-publish-secret-name: px-secret
      csi.storage.k8s.io/node-publish-secret-namespace: kube-system
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    allowVolumeExpansion: true
    ```


### Encrypt your volumes per PVC

You can encrypt volumes by allowing your users to specify encryption keys in their PVCs. Using this method, each PVC will use its own key for encryption. Follow the steps below to create two PVCs which use different passphrases for encryption:

#### Step 1: Create two Kubernetes secrets which will be used for two different PVCs

Enter the following `kubectl create secret generic` command for your first PVC, specifying your own passphrase in `mysecret-passcode-for-encryption` which encrypts the PVC:

```text
kubectl create secret generic volume-secrets-1 -n kube-system --from-literal=mysql-pvc-secret-key-1=mysecret-passcode-for-encryption-1
```

Enter the same command for your second PVC, and optionally, specify a different secret name and a different namespace. This example places both PVCs in a single namespace:

```text
kubectl create secret generic volume-secrets-2 -n kube-system --from-literal=mysql-pvc-secret-key-2=mysecret-passcode-for-encryption-2
```

#### Step 2: Create two CSI Kubernetes secrets that will point to their own encryption Kubernetes secrets

Enter the following `kubectl create secret generic` command for your first PVC, specifying the following options:

  * This secret's name, `mysql-pvc-1` in this example
  * The `-n` option with the same namespace which the kubernetes secret points to`kube-system`, in this example
  * The `--from-literal=SECRET_NAME=` option and the name of the kubernetes secret this must point to 
  * The `--from-literal=SECRET_KEY=` option and the key inside the second secret that houses the encryption key
  * The `--from-literal=SECRET_CONTEXT=` option and the secret's namespace


```text
kubectl create secret generic mysql-pvc-1 -n kube-system --from-literal=SECRET_NAME=volume-secrets-1 --from-literal=SECRET_KEY=mysql-pvc-secret-key-1 --from-literal=SECRET_CONTEXT=portworx
```

Enter the same command for your second PVC, but specify a different secret name and optionally, a different namespace:

```text
kubectl create secret generic mysql-pvc-2 -n kube-system --from-literal=SECRET_NAME=volume-secrets-2 --from-literal=SECRET_KEY=mysql-pvc-secret-key-2 --from-literal=SECRET_CONTEXT=portworx
```

#### Step 3: Create a CSI storage class for encrypted PVCs

Create a StorageClass CRD, specifying the `${pvc.name}` and `${pvc.namespace}` template variables:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: px-csi-db-encrypted-k8s
provisioner: pxd.portworx.com
parameters:
  repl: "3"
  secure: "true"
  io_profile: auto
  io_priority: "high" 
  cow_ondemand: "false"
  csi.storage.k8s.io/provisioner-secret-name: ${pvc.name}
  csi.storage.k8s.io/provisioner-secret-namespace: ${pvc.namespace}
  csi.storage.k8s.io/node-publish-secret-name: ${pvc.name}
  csi.storage.k8s.io/node-publish-secret-namespace: ${pvc.namespace}
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```

#### Step 4: Create encrypted PVCs

Create two encrypted PVCs, one for each of the secrets you created in the preceding steps:

1. Create the `mysql-pvc-1` PVC that uses the passcode you created in **Step 1: Create two kubernetes secrets which is used for two different PVCs**. In the example, that passcode is `mysecret-passcode-for-encryption-1`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: mysql-pvc-1
        namespace: kube-system
      spec:
        storageClassName: portworx-sc
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 2Gi
      ```

2. Create the `mysql-pvc-2` PVC that uses the passcode you created in **Step 1: Create two kubernetes secrets which will be used for two different PVCs**. In the example, that passcode is `mysecret-passcode-for-encryption-2`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: mysql-pvc-2
        namespace: kube-system
      spec:
        storageClassName: portworx-sc
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 2Gi
      ```

The templatized parameters in the CSI storage class point to the name and namespace of the PVC itself. This ensures that each PVC requires a separate Kubernetes secret of the same name in the same namespace. In this way, each PVC gets encrypted with its own passphrase.
