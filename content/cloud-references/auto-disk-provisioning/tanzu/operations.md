---
title: Operate Portworx on VMware Tanzu
description: Learn how to operate Portworx cluster on VMware Tanzu.
keywords: VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: Operate
weight: 400
noicon: true
---

This page describes operations for Portworx clusters on VMWare Tanzu.

## Scale down clusters

A Portworx cluster on Tanzu can contain a mixture of storage and storageless nodes.

When scaling down a Tanzu cluster, do not scale down the cluster size lower than the total number of storage nodes in the Portworx cluster.

You can find out the number of storage nodes by running `pxctl status` on any PX node. This will list all nodes in the cluster and mention which ones are storage nodes.

To control the number of storage nodes in the cluster, use the `spec.cloudStorage.maxStorageNodesPerZone` or `spec.cloudStorage.maxStorageNodes` configuration in [`StorageCluster`](/reference/crd/storage-cluster/).

## Pool expansion limitations

In vSphere with Tanzu and the Tanzu Kubernetes Grid environment, pool expansion is **only supported by adding a new disk**, and that new disk must be the same size as the original disk. For example, if the pool has disk size of 150GB, it can only be expanded by multiples of 150GB.

If you attempt to expand by less than the size of the existing disk, the pool will expand by the size of the existing disk. For example, if you run the following command on a pool with a disk size of 150GB, the operation will succeed:

```text
pxctl service pool expand -o add-disk -s 250 -u <pool_uuid>
```
```output
Request to expand pool: <pool_uuid> to size: 250 using operation: add-disk
Pool resize triggered successfully.
```

However, the disk will actually expand to 300GB:

```text
pxctl service pool show
```
```output
PX drive configuration:
Pool ID: 0 
    UUID:  <pool_uuid> 
    ... 
    Size: 300 GiB
```

### Unsupported expansion operations

The expansion operations `resize` and `auto` are not supported on vSphere with Tanzu and the Tanzu Kubernetes Grid environment.

For example, if you run the following command with `-o resize-disk`, the output indicates that the resize has triggered successfully:

```text
pxctl service pool expand -o resize-disk -s 300 -u <pool_uuid>
```
```output
Request to expand pool: <pool_uuid> to size: 300 using operation: resize-disk
Pool resize triggered successfully.
```

However, when you view the pool status, it shows that the operation has failed:

```text
pxctl service pool show
```
```output
PX drive configuration:
Pool ID: 0 
    UUID:  <pool_uuid> 
    ...
    LastOperation  OPERATION_RESIZE 
        Status:  OPERATION_FAILED 
        Message: failed to resize cloud drive to: 300 due to: Operation: Drive Resize is not supported. Reason: vSphere limitations
```

Similarly, using `-o auto` will indicate that the resize has triggered successfully:

```text
pxctl service pool expand -o auto -s 300 -u <pool_uuid>
```
```output
Request to expand pool: <pool_uuid> to size: 300 using operation: auto
Pool resize triggered successfully.
```

However, the pool status shows the same failure:

```text
pxctl service pool show
```
```output
PX drive configuration:
Pool ID: 0 
    UUID:  <pool_uuid> 
    ...
    LastOperation  OPERATION_RESIZE 
        Status:  OPERATION_FAILED 
        Message: failed to resize cloud drive to: 300 due to: Operation: Drive Resize is not supported. Reason: vSphere limitations
```