---
title: Volume Lifecycle Basics
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 2
---

## Create and use persistent volumes

To enable CSI for a StorageClass, set the `provisioner` value to `pxd.portworx.com`:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-csi-sc
provisioner: pxd.portworx.com
parameters:
  repl: "1"
```

To create a PersistentVolumeClaim based on your CSI-enabled StorageClass, reference the StorageClass you created above with the `storageClassName` parameter:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: px-mysql-pvc
spec:
   storageClassName: portworx-csi-sc
   accessModes:
     - ReadWriteOnce
   resources:
     requests:
       storage: 2Gi
```

Once you've created a storage class and PVC, you can create a volume as part of a deployment by referencing the PVC. This example creates a MySQL deployment referencing the `px-mysql-pvc` PVC you created in the step above:

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
        version: "1"
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: px-mysql-pvc
 ```

<!--

### Mixing hardcoded and template values


You can mix hardcoded and template values in your StorageClass.

Hardcode the secret name, but use template values for the namespace to allow users to create PVCs on any namespace using your hardcoded secret.

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-csi-sc
provisioner: pxd.portworx.com
parameters:
  repl: "1"
  csi.storage.k8s.io/provisioner-secret-name: px-secret
  csi.storage.k8s.io/provisioner-secret-namespace: ${pvc.namespace}
```

Hardcode the namespace but use template values for the secret to allow users to specify their own secret in the PVC, but only on the namespace you specify:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-csi-sc
provisioner: pxd.portworx.com
parameters:
  repl: "1"
  csi.storage.k8s.io/provisioner-secret-name: ${pvc.name}
  csi.storage.k8s.io/provisioner-secret-namespace: portworx
```
-->

## Create shared CSI-enabled volumes

You can create shared CSI-enabled volumes using one of two methods:

* Using a PVC's AccessMode parameter
* Using StorageClass parameters

### Sharedv4

In your PVC, if you use `ReadWriteMany`, Portworx defaults to a `Sharedv4` volume type.

* In your SC, if you use the parameter `Sharedv4=true` and `ReadWriteMany` in your PVC, Portworx uses the Sharedv4 volume type.
* In your SC, if you use the parameter `Shared=true` and `ReadWriteMany` in your PVC, Portworx uses the Shared volume type.


## Clone volumes with CSI

You can clone CSI-enabled volumes, duplicating both the volume and content within it.

1. Enable the `VolumePVCDataSource` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/):

      ```text
      --feature-gates=VolumePVCDataSource=true
      ```

2. Create a PVC that references the PVC you wish to clone, specifying the `dataSource` with the kind and name of the target PVC you wish to clone. The following spec creates a clone of the `px-mysql-pvc` PVC in a YAML file named `clonePVC.yaml`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
         name: clone-of-px-mysql-pvc
      spec:
         storageClassName: portworx-csi-sc
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 2Gi
         dataSource:
          kind: PersistentVolumeClaim
          name: px-mysql-pvc
      ```

3. Apply the `clonePVC.yaml` spec to create the clone:

      ```text
      kubectl apply -f clonePVC.yaml
      ```

## Migration to CSI PVCs

Currently, you cannot migrate or convert PVCs created using the native Kubernetes driver to the CSI driver. However, this is not required and both types of PVCs can co-exist on the same cluster.


## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
