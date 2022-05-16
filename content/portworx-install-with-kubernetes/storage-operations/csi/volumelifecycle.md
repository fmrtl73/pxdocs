---
title: Volume Lifecycle Basics
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 200
---

## Create and use persistent volumes

Create and use volumes with CSI by configuring specs you create for your storage class, PVC, and volumes:

1. Enable CSI for a StorageClass by setting the `provisioner` value to `pxd.portworx.com`:

    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-csi-sc
    provisioner: pxd.portworx.com
    parameters:
      repl: "1"
    ```

2. Create a PersistentVolumeClaim based on your CSI-enabled StorageClass by referencing the StorageClass you created above with the `storageClassName` parameter:

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

3. Create a volume by referencing the PVC. This example creates a MySQL deployment referencing the `px-mysql-pvc` PVC you created in the step above:

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

## Create shared CSI-enabled volumes

Create shared CSI-enabled volumes by configuring the specs for you create for your storage class and PVCs:

1. Create a StorageClass `storageclass.yaml` file and apply it:

    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-csi-sc
    provisioner: pxd.portworx.com
    parameters:
      repl: "1"
    ```

2. Apply the `storageclass.yaml` file:

    ```text
    kubectl apply -f storageclass.yaml
    ```

3. Create a sharedv4 PVC by creating the following `shared-pvc.yaml` file:

    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: px-mysql-pvc
    spec:
      storageClassName: portworx-csi-sc
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 2Gi
    ```

4. Apply the `shared-pvc.yaml` file:

    ```text
    kubectl apply -f shared-pvc.yaml
    ```

## Clone volumes with CSI

You can clone CSI-enabled volumes, duplicating both the volume and content within it.

1. Create a PVC that references the PVC you wish to clone, specifying the `dataSource` with the kind and name of the target PVC you wish to clone. The following spec creates a clone of the `px-mysql-pvc` PVC in a YAML file named `clonePVC.yaml`:

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

2. Apply the `clonePVC.yaml` spec to create the clone:

      ```text
      kubectl apply -f clonePVC.yaml
      ```

## Migration to CSI PVCs

Currently, you cannot migrate or convert PVCs created using the native Kubernetes driver to the CSI driver. However, this is not required and both types of PVCs can co-exist on the same cluster.


## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
