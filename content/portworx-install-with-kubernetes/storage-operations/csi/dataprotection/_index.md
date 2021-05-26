---
title: Data Protection and Snapshots
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, snapshot
description: This explains how you can protect your data against failures with Stork or CSI Snapshotting
weight: 3
---

## Take snapshots of CSI-enabled volumes

For Kubernetes 1.17+, CSI Snapshotting is in beta is supported by the Portworx CSI Driver.

Given you already have a [CSI PVC and StorageClass](/portworx-install-with-kubernetes/storage-operations/csi/volumelifecycle), complete the following steps to create and restore a CSI VolumeSnapshot.

1. Create a VolumeSnapshotClass, specifying the following: 
    * The `snapshot.storage.kubernetes.io/is-default-class: "true"` annotation
    * The `csi.storage.k8s.io/snapshotter-secret-name` parameter with your encryption and/or authorization secret
    * The `csi.storage.k8s.io/snapshotter-secret-namespace` parameter with the namespace your secret is in

      ```text
      apiVersion: snapshot.storage.k8s.io/v1beta1
      kind: VolumeSnapshotClass
      metadata:
        name: px-csi-snapclass
        annotations:
          snapshot.storage.kubernetes.io/is-default-class: "true"
      driver: pxd.portworx.com
      deletionPolicy: Delete
      parameters:
        csi.storage.k8s.io/snapshotter-secret-name: px-user-token
        csi.storage.k8s.io/snapshotter-secret-namespace: portworx
      ```

2. Create a VolumeSnapshot:

      ```text  
      apiVersion: snapshot.storage.k8s.io/v1beta1
      kind: VolumeSnapshot
      metadata:
        name: px-csi-snapshot
      spec:
        volumeSnapshotClassName: px-csi-snapclass
        source:
          persistentVolumeClaimName: px-mysql-pvc
      ```

3. Restore from a VolumeSnapshot:

      ```text
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: px-csi-pvc-restored 
      spec:
        storageClassName: portworx-csi-sc
        dataSource:
          name: px-csi-snapshot
          kind: VolumeSnapshot
          apiGroup: snapshot.storage.k8s.io
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
      ```


See the [Kubernetes-CSI snapshotting documentation](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) for more examples and documentation. 

## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).