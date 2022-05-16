---
title: Secure your volumes 
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, security
description: This page explains how you can start securing your volume with PX Security
weight: 400
---

# Encryption with CSI

For information about how to encrypt PVCs on CSI using Kubernetes secrets, see the [Encrypting PVCs on CSI with Kubernetes Secrets](/key-management/kubernetes-secrets/pvc-encryption-using-csi/) section of the Portworx documentation.


# Authorization and Authentication

You can secure your CSI-enabled volumes with token-based authorization. In using token-based authorization, you create secrets containing your token credentials and specify them in your StorageClass in one of two ways:

* Using hardcoded values
* Using template values

You can also mix these two methods to form your own hybrid approach.

### Using hardcoded values

This example secures a storage class by specifying hardcoded values for the token and namespace. Users who create PVCs based on this StorageClass will always have their PVCs use the `px-user-token` StorageClass under the `portworx` namespace.

1. Find or create your token secret:

    For operator installs, a user token is automatically created and refreshed under `px-user-token` in your `StorageCluster` namespace.
    ```text
    USER_TOKEN=$(kubectl get secrets px-user-token -n portworx -o json | jq -r '.data["auth-token"]' | base64 -d)
    ```

    For all other configurations, [create your own token secret](/cloud-references/security/kubernetes/shared-secret-model/generating-tokens/):
    ```text
    kubectl create secret generic px-user-token \
      -n portworx --from-literal=auth-token=$USER_TOKEN
    ```

2. Modify the storageClass, adding the following parameters:

    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-csi-sc
    provisioner: pxd.portworx.com
    parameters:
      repl: "1"
      csi.storage.k8s.io/provisioner-secret-name: px-user-token
      csi.storage.k8s.io/provisioner-secret-namespace: portworx
      csi.storage.k8s.io/node-publish-secret-name: px-user-token
      csi.storage.k8s.io/node-publish-secret-namespace: portworx
    ```

### Using template values

This example secures a storage class by hardcoding the token and namespace. Users who create PVCs based on this StorageClass can have have their PVCs use the StorageClass they specified in the annotation of their PVC, and the namespace they specified in their PVC.

1. Modify the storageClass, adding the following parameters:

    * `csi.storage.k8s.io/provisioner-secret-name: ${pvc.name}`
    * `csi.storage.k8s.io/provisioner-secret-namespace: ${pvc.namespace}`
    * `csi.storage.k8s.io/node-publish-secret-name: ${pvc.name}`
    * `csi.storage.k8s.io/node-publish-secret-namespace: ${pvc.namespace}`

       ```text
       kind: StorageClass
       apiVersion: storage.k8s.io/v1
       metadata:
         name: portworx-csi-sc
       provisioner: pxd.portworx.com
       parameters:
         repl: "1"
         csi.storage.k8s.io/provisioner-secret-name: ${pvc.name}
         csi.storage.k8s.io/provisioner-secret-namespace: ${pvc.namespace}
         csi.storage.k8s.io/node-publish-secret-name: ${pvc.name}
         csi.storage.k8s.io/node-publish-secret-namespace: ${pvc.namespace}
       ```
2. Make sure your user also creates a secret in their PVC's namespace with the same name as the PVC. For example, a PVC named "px-mysql-pvc" must have an associated secret named "px-mysql-pvc". Enter the following command to create the secret:

    ```text
    kubectl create secret generic px-mysql-pvc -n portworx --from-literal=auth-token=$USER_TOKEN
    ```