---
title: Shared and Sharedv4 Volumes
linkTitle: Shared Volumes
description: Explanation on Portworx Shared and Sharedv4 volumes to allow multiple containers access to one volume
keywords: portworx, PX-Developer, container, Shared Volume, Sharedv4 Volume, NFS, storage
hidden: true
---

Through shared and sharedv4 volumes (also known as a **global namespace**), a single volume’s filesystem is concurrently available to multiple containers running on multiple hosts.

{{<info>}}
**Note:**

* You do not need to use shared/sharedv4 volumes to have your data accessible on any host in the cluster. Any Portworx volumes can be exclusively accessed from any host as long as they are not simultaneously accessed. Shared volumes are for providing simultaneous (concurrent or shared) access to a volume from multiple hosts at the same time.
* You do not necessarily need a replication factor of greater than 1 on your volume in order for it to be shared. Even a volume with a replication factor of 1 can be shared on as many nodes as there are in your cluster.
* IOPS might be misleading due to batching of small blocksize I/Os into a larger one before I/O reaches the `pxd` device, especially when using sharedV4 volumes. Bandwidth is more consistent.
{{</info>}}

A typical pattern is for a single container to have one or more volumes. Conversely, many scenarios would benefit from multiple containers being able to access the same volume, possibly from different hosts. Accordingly, the shared volume feature enables a single volume to be read/write accessible by multiple containers. Example use cases include:

* A technical computing workload sourcing its input and writing its output to a shared volume.
* Scaling a number of Wordpress containers based on load while managing a single shared volume.
* Collecting logs to a central location.

{{<info>}}
**Note:**
Usage of shared or sharedv4 volumes for databases is not recommended because they have a small metadata overhead. Additionally, typical databases do not support concurrent writes to the underlying database at the same time.
{{</info>}}

The difference between shared and sharedv4 volumes is the underlying protocol that is used to share this global namespace across multiple hosts.

## Sharedv4 failover and failover strategy

When the node which is exporting the sharedv4 or sharedv4 service volume becomes unavailable, there is a sharedv4 failover. After failover, the volume is exported from another node which has a replica of the volume.

Failover is handled slightly differently for sharedv4 volumes than for sharedv4 service volumes:

* When a sharedv4 volume fails over, all of the application pods are restarted.
* For sharedv4 service volumes, only a subset of the pods need to be restarted. These are the pods that were running on the 2 nodes involved in failover: the node that became unavailable, and the node that started exporting the replica of the volume. The pods running on the other nodes do not need to be restarted.

The _failover strategy_ determines how quickly the failover will start after detecting that the node exporting the volume has become unavailable. The `normal` strategy waits for a longer duration than the `aggressive` strategy.

### Sharedv4 volumes

The default failover strategy for sharedv4 volumes is `normal`. This gives the unavailable node more time to come back up after a transient issue. If the node comes back up during the grace period allowed by the `normal` failover strategy, there is no need to restart the application pods.

If an application with a sharedv4 volume is able to recover quickly after a restart, it may be more appropriate to use the `aggressive` failover strategy even for a sharedv4 volume.

### Sharedv4 service volumes

The default failover strategy for sharedv4 service volumes is `aggressive`, because these volumes are able to fail over without restarting all the application pods.

These defaults can be changed in the following ways:

* [Setting a value for `sharedv4_failover_strategy` in StorageClass](/portworx-install-with-kubernetes/storage-operations/create-pvcs/create-sharedv4-pvcs/) before provisioning a volume.
* Using a `pxctl volume update` command if a volume has been provisioned already. For example:
  ```text
  pxctl volume update --sharedv4_failover_strategy=normal <volume_ID>
  ```