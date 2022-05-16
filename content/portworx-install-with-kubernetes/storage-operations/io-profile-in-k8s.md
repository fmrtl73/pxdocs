---
title: Configure IO profiles for Portworx volumes on Kubernetes
linkTitle: Configure IO profiles
keywords: stork, kubernetes, k8s
description: Learn how to apply IO profiles to Portworx volumes on Kubernetes.
weight: 90000
---

Portworx volumes on Kubernetes can use different IO profiles to optimize the Portworx volumes' performance for specific use-cases. For information about what IO profiles are, as well as the different profiles available, see the [IO profiles](/concepts/io-profiles/) concept section. 

{{<info>}}
**NOTE:** 

* By default, Portworx uses an `auto` IO profile, which automatically applies an IO profile that is most appropriate for the data patterns it sees.
* You can also configure these profiles manually using `pxctl`. For information about performance tuning using `pxctl`, visit the [Performance tuning](/install-with-other/operate-and-maintain/performance-and-tuning/tuning/) section of the documentation.
{{</info>}}

## Manually configure IO profiles

You can direct a group of volumes to use a specific IO profile by defining it at the storageClass level and referencing that storageClass in your PVCs:

1. Create and configure a storageClass as desired. Add the `parameters.io_profile` field with the IO profile you want to specify:
   
    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-io-profile
    provisioner: kubernetes.io/portworx-volume
    parameters:
    repl: "2"
    io_profile: "db_remote"
    ```

2. In your PVCs, reference the storageClass you just created to apply your IO profile to volumes: 
   
    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: px-db-remote-pvc
      annotations:
        volume.beta.kubernetes.io/storage-class: portworx-io-profile
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    ```