---
title: Using Pre-provisioned Volumes
weight: 1000
keywords: pre-provisioned volumes, Kubernetes, k8s
description: Learn how to use a pre-provisioned Portworx volume in Kubernetes
series: k8s-vol
---

This document describes how to use a pre-provisioned volume in your Kubernetes cluster.

## Creating a Portworx volume using pxctl

First create a volume using the Portworx CLI. On one of the nodes with Portworx installed, run the following command:

```text
/opt/pwx/bin/pxctl volume create testvol --size 2
```

For more details on creating volumes using `pxctl`, see [Concepts](/concepts).

Alternatively, you can also use [snapshots](/reference/cli/snapshots/) that you previously created.

## Using the Portworx volume

Once you have a Portworx volume, you can use it in 2 different ways:

### 1. Using the Portworx volume directly in a pod

You can create a pod that directly uses a Portworx volume as follows:

```text
apiVersion: v1
kind: Pod
metadata:
   name: nginx-px
spec:
   containers:
   - image: nginx
     name: nginx-px
     volumeMounts:
     - mountPath: /test-portworx-volume
       name: testvol
   volumes:
   - name: testvol
     # This Portworx volume must already exist.
     portworxVolume:
       volumeID: testvol
```

{{<info>}}**NOTE:** The _name_ and _volumeID_ above must be the same and should be the name of the Portworx volume created using pxctl.{{</info>}}

### 2. Using the Portworx volume by creating a PersistentVolume & PersistentVolumeClaim

**Creating PersistentVolume**

First create a `PersistentVolume` that references the Portworx volume. Following is an example spec.

```text
apiVersion: v1
kind: PersistentVolume
metadata:
  name: testvol
spec:
  capacity:
    storage: 2Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: testvol-pvc
    namespace: default
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  portworxVolume:
    volumeID: <PX_VOLUME_ID>
```

{{<info>}}**NOTE:** The preceding `PersistentVolume` example references an existing Portworx volume called `testvol` that was created using `pxctl`.{{</info>}}

#### Creating PersistentVolumeClaim

Now create a `PersistentVolumeClaim` that will claim the previously created volume. Following is an example spec.

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: testvol-pvc
spec:
  selector:
    matchLabels:
      name: testvol
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: testvol
```

{{<info>}}**NOTE:** If you are planning to use the `PersistentVolumeClaim` in a pod in a non-default namespace, you must create the `PersistentVolumeClaim` in that namespace.
{{</info>}}

#### Creating a pod using the PersistentVolumeClaim

Now you can create a pod that references the `PersistentVolumeClaim` that you created. Following is an example.

```text
apiVersion: v1
kind: Pod
metadata:
   name: nginx-px
spec:
  containers:
  - image: nginx
    name: nginx-px
    volumeMounts:
    - mountPath: /test-portworx-volume
      name: testvol
  volumes:
  - name: testvol
    persistentVolumeClaim:
      claimName: testvol-pvc
```

{{<info>}}**NOTE:** To access PV/PVCs with a non-root user, refer to [Access via Non-Root Users](/portworx-install-with-kubernetes/storage-operations/create-pvcs/access-via-non-root-users){{</info>}}
