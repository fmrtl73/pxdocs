---
title: Data Protection and Snapshots
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, snapshot
description: This explains how you can protect your data against failures with Stork or CSI Snapshotting
weight: 300
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/dataprotection/
---
## Setup CSI Volume Snapshotting 

In order to use VolumeSnapshots with the Portworx CSI Driver and Portworx Operator, you must complete the following steps:

1. Install the VolumeSnapshot CRDs:

    * For Kubernetes 1.20+, use v1:

      ```text
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v4.1.1/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v4.1.1/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v4.1.1/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
      ```

    * For Kubernetes 1.17-1.19, use v1beta1:

      ```text
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v3.0.3/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v3.0.3/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v3.0.3/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
      ```
    

2. Install the Snapshot Controller:

    * For Kubernetes 1.20+, use v4:

      ```text
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v4.1.1/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v4.1.1/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
      ```

    * For Kubernetes 1.17-1.19, use v3:

      ```text
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v3.0.3/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/v3.0.3/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
      ```

## Take snapshots of CSI-enabled volumes

For Kubernetes 1.17+, CSI Snapshotting is in beta is supported by the Portworx CSI Driver.

Given you already have a [CSI PVC and StorageClass](/operations/operate-kubernetes/storage-operations/csi/volumelifecycle), complete the following steps to create and restore a CSI VolumeSnapshot.

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
