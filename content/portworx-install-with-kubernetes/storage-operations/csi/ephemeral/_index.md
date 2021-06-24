---
title: Use Ephemeral volumes
keywords: csi, ephemeral, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains how to use ephemeral volumes with the Portworx CSI Driver
weight: 5
---

## Generic Ephemral volumes

Generic ephemeral volumes is a newer feature and is in beta (enabled by default) as of Kubernetes 1.21. These volumes also work with typical storage operations such as snapshotting, cloning, resizing, and storage capacity tracking.

The following steps will allow you to create a generic ephemeral volume with the Portworx CSI Driver.

1. Create a StorageClass spec that references the Portworx CSI Driver provisioner as seen below in a yaml file named `portworx-sc.yaml`:

      ```text
      kind: StorageClass
      apiVersion: storage.k8s.io/v1
      metadata:
        name: portworx-csi-sc
      provisioner: pxd.portworx.com
      parameters:
        repl: "1"
      ```

2. Apply the `portworx-sc.yaml` spec to create the Portworx CSI Driver StorageClass:

      ```text
      kubectl apply -f portworx.sc.yaml
      ```

3. Create a pod spec that uses a Portworx CSI Driver StorageClass, declaring the ephemeral volume as seen below in a yaml file named `ephemeral-volume-pod.yaml`:

      ```text
      kind: Pod
      apiVersion: v1
      metadata:
        name: my-app
      spec:
        containers:
          - name: my-frontend
            image: busybox
            volumeMounts:
            - mountPath: "/scratch"
              name: scratch-volume
            command: [ "sleep", "1000000" ]
        volumes:
          - name: scratch-volume
            ephemeral:
              volumeClaimTemplate:
                metadata:
                  labels:
                    type: my-frontend-volume
                spec:
                  accessModes: [ "ReadWriteOnce" ]
                  storageClassName: "scratch-storage-class"
                  resources:
                    requests:
                      storage: 1Gi
      ```

4. Apply the `ephemeral-volume-pod.yaml` spec to create the pod with a generic ephemeral volume:

      ```text
      kubectl apply -f ephemeral-volume-pod.yaml
      ```

## Migration to CSI PVCs

Currently, you cannot migrate or convert PVCs created using the native Kubernetes driver to the CSI driver. However, this is not required and both types of PVCs can co-exist on the same cluster.


## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
