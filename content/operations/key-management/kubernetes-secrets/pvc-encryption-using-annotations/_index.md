---
title: Encrypting PVCs using annotations with Kubernetes Secrets
weight: 100
keywords: Encrypt PVC, annotations, kubernetes, k8s
description: Instructions on using Kubernetes Secrets with Portworx for encrypting PVCs using annotations
noicon: true
series: kubernetes-secret-uses
series2: k8s-pvc-enc
hidden: true
aliases:
    - /key-management/kubernetes-secrets/pvc-encryption-using-annotations/
---
{{< content "shared/key-management-intro.md" >}}

{{<info>}}
**NOTE:** Supported from {{< pxEnterprise >}} 1.4 onwards
{{</info>}}

[Encryption at Storage Class level](/operations/key-management/kubernetes-secrets/pvc-encryption-using-storageclass) does not allow using different secret keys for different PVCs. It also does not provide a way to disable encryption for certain PVCs that are using the same secure storage class. Encryption at PVC level will override the encryption options from Storage Class.

PVC level encryption is achieved using following PVC annotations:

- `px/secure` - Boolean which tells to secure the PVC or not
- `px/secret-name` - Name of the secret used to encrypt
- `px/secret-namespace` - Namespace of the secret (Kubernetes Secrets only)
- `px/secret-key` - Key to be used in the secret (Kubernetes Secrets only)

### Encryption using cluster wide secret

#### Step 1: Create cluster wide secret key
A cluster wide secret key is a common key that points to a secret value/passphrase which can be used to encrypt all your volumes.

Create a cluster wide secret in Kubernetes, if not already created:

```text
kubectl -n portworx create secret generic px-vol-encryption \
  --from-literal=cluster-wide-secret-key=<value>
```

Note that the cluster wide secret has to reside in the `px-vol-encryption` secret under the `portworx` namespace.

Now you have to give Portworx the cluster wide secret key, that acts as the default encryption key for all volumes.

```text
PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl secrets set-cluster-key \
  --secret cluster-wide-secret-key
```

#### Step 2: Create the secure PVC
If your Storage Class does not have the `secure` flag set, but you want to encrypt the PVC using the same Storage Class, then create the PVC as below:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-pvc
  annotations:
    px/secure: "true"
spec:
  storageClassName: portworx-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

As there is no `px/secret-name` annotation specified, Portworx will default to the cluster wide secret to encrypt this PVC. If the cluster wide secret is not set, the volume creation will fail until the key is set.

Similar to the above example, if you want to use a Storage Class with `secure` parameter set, but do not want to encrypt a certain PVC, then set the `px/secure` annotation to `false`.

{{<info>}}
If you are running Kubernetes version older than 1.9.4 (or < 1.8.9 in Kubernetes 1.8), then the PVC name has to be in `ns.<namespace_of_pvc>-name.<identifier_for_pvc>` format to use the PVC-level encryption feature.
{{</info>}}


### Encryption using custom secret key

#### Kubernetes secrets
You can encrypt your PVC using a custom secret as follows:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
  annotations:
    px/secret-name: volume-secrets
    px/secret-namespace: portworx
    px/secret-key: mysql-pvc
spec:
  storageClassName: portworx-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

The encrypted PVC will use the key `mysql-pvc` under the Kubernetes secret `volume-secrets` in `portworx` namespace. If the secret key is not present, then the volume creation will fail until the key is created.

From the annotations in the above PVC, only `px/secret-name` is mandatory.
- If you do not specify the `px/secret-namespace`, Portworx will look for the secret in the PVC's namespace.
- If you do not specify the `px/secret-key`, Portworx will look for a key with the PVC name.

By default, Portworx has `get` and `list` permissions for Kubernetes secrets from all the namespaces. In the above example, you can replace the `px/secret-namespace` annotation with a namespace of your choice where you have created the Kubernetes secret.
