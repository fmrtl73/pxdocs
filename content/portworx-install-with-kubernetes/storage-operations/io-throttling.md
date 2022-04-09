---
title: Application I/O Control
keywords: stork, kubernetes, k8s
description: 
weight: 910
---

Portworx volumes are created from a common storage pool and share their available IOPS and bandwidth between all other provisioned Portworx volumes. If an application's volumes start consuming too many resources, it can become a "noisy neighbor" and reduce the I/O and network bandwidth available to other volumes in the pool. 

The Portworx Application I/O Control feature provides you with a method for controlling an individual Portworx volume’s IOPS or bandwidth usage from within Kubernetes. You can define quotas through a StorageClass which prevent any applications from using excessive bandwidth. When you need to adjust the bandwidth, you can do so in real-time by updating the values in the StorageClass. 

<!-- For example, if you're performing testing or staging operations in one namespace in a cluster, a bug or misconfiguration in your test applications could impact volume performance for production applications in another namespace. You can prevent this by using the Application I/O Control StorageClass parameters to cap your test application volume bandwidth. -->

In addition to defining parameters in a StorageClass, you can set Application I/O Control values for volumes using the `pxctl` command line interface. 


{{<info>}}
**NOTE:** By default, this feature is not enabled.
{{</info>}}

## Prerequisites

* Application I/O Control is supported for kernel 4.6 or higher.
* Only the cgroup mount path `/sys/fs/cgroup/blkio` is supported. This is typically set by `mount -t cgroup -o blkio none /sys/fs/cgroup/blkio`. Uncommon distributions could have a different cgroup mount path.

## Create volumes

{{<info>}}
**NOTE:**

* You can set a maximum rate for IOPS and a maximum rate for bandwidth at the same time. However, you cannot set a read parameter for both IOPS and bandwidth at the same time or a write parameter for both at the same time. For example, setting `--max_iops 1024,off --max_bandwidth off,2` will work, but setting `--max_iops 1024,off --max_bandwidth 2,off` will prompt error. 

* Once you set Application I/O Control values, you can check alerts or logs for messages indicating that Application I/O Control is not enabled. For example: `IO throttling supported from kernel v4.6, this node has kernel v3.10` indicates that Application I/O Control is not supported.

* IOPS might be misleading due to batching of small blocksize IOs into a larger one before IO reaches the pxd device, especially when using sharedV4 volumes. Bandwidth throttling is more consistent.
{{</info>}}

### Apply using a StorageClass

You can apply Application I/O Control through a StorageClass.

To configure IOPS throttling in a StorageClass, use the following format:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-io-profile
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "2"
  io_profile: "db_remote"
  io_throttle_rd_iops: "1024"
  io_throttle_wr_iops: "1024"
```

To configure bandwidth throttling in a StorageClass, use the following format:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-io-profile
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "2"
  io_profile: "db_remote"
  io_throttle_rd_bw: "10"
  io_throttle_wr_bw: "10"
```

You can also configure read IOPS throttling and write bandwidth throttling, or the inverse. For example:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-io-profile
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "2"
  io_profile: "db_remote"
  io_throttle_rd_iops: "1024"
  io_throttle_wr_bw: "10"
```

{{<info>}}
**NOTE:**The minimum value for `rd_bw` and `wr_bw` is `1`, and the minimum value for `rd_iops` and `wr_iops` is `256`.
{{</info>}}

### Create using pxctl

When you create volumes using `pxctl`, you can specify the `--max_bandwidth` and `--max_iops` flags to restrict the storage resources that volume uses:

```text
pxctl volume create --max_bandwidth <ReadBW>,<WriteBW> <volname/volid> 
```
```text
pxctl volume create --max_iops <ReadIOPS>,<WriteIOPS> <volname/volid> 
```

{{<info>}}
**NOTE:**The minimum value for `--max_bandwidth` is `1`, and the minimum value for `--max_iops` is `256`.
{{</info>}}

## Update volumes using pxctl

If you want to add Application I/O Control to an existing volume, or change its throttling values, you can do so using the `--max_bandwidth` and `--max_iops` flags with the `pxctl volume update` commands:

```text
pxctl volume update --max_bandwidth <ReadBW>,<WriteBW> <volname/volid> 
```
```text
pxctl volume update --max_iops <ReadIOPS>,<WriteIOPS> <volname/volid> 
```

{{<info>}}
**NOTE:** If you want to limit only read or write bandwidth/IOPS traffic, set the other value to `off`. 

For example: Set read throttling for an existing volume to 1024 IOPS:

```text
pxctl volume update --max_iops 1024,off exampleVolume
```
{{</info>}}

## Remove throttling

If you want to remove throttling from a volume, set the value of the `--max_bandwidth` and `--max_iops` flags to `off` for any throttling you wish to remove: 

```text
pxctl volume update --max_bandwidth off,<WriteBW> <volname/volid>
```
```text
pxctl volume update --max_iops off,off <volname/volid>
```