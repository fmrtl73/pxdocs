---
title: RelaxedReclaim
weight: 800
keywords: snapshots, scheduled, kubernetes, k8s, relaxed, RelaxedReclaim
description: Employ a reclaim policy that spreads load over time. 
---

When Portworx deletes snapshots and volumes, it also deletes their replicas. In some scenarios, you may delete a large number of replicas at once. These delete requests can overwhelm the underlying filesystem, causing high filesystem latencies and reducing I/O performance. Using RelaxedReclam, you can stage snapshot and volume replica delete operations in a queue and spread them out over time, giving the filesystem enough bandwidth to handle front-end I/O and lowering filesystem latencies. 

RelaxedReclaim works by allowing you to specify a delay, in seconds, between each operation in the queue. Whenever you delete a volume or snapshot, Portworx places a delete operation for each replica targeted by the operation in the queue. The total duration of the RelaxedReclaim operation then becomes equal to the number of replicas in a pool times the rate in seconds at which Portworx deletes them. 

For example, a snapshot policy contains definitions for the snapshot frequency and how many snapshots are retained. Once the number of snapshots retained reaches the limit defined in the policy, Portworx deletes an old snapshot every time a new one is taken. On clusters that have a large number of devices, and if all share the same snapshot frequency, Portworx will send a large number of concurrent delete requests for all the replicas to the storage pool. RelaxedReclaim paces these snapshot replica deletes, preserving filesystem performance. 

## Enable or reconfigure RelaxedReclaim

The `relaxedreclaim-delete-seconds` flag sets the time window of a single volume delete. For example, a `relaxedreclaim-delete-seconds` value of `60` means that 1 volume/snapshot replica will be deleted every 60 seconds. By default, the RelaxedReclaim value set to zero. To enable or reconfigure it, you must change the value as a cluster option. 

Enter the following `pxctl` command:

```text
pxctl cluster options update --relaxedreclaim-delete-seconds 5
```
<!-- what's a good example value? -->

Once you've applied the cluster option, Portworx will stage any new snapshot and volume deletions and delete them from the pool at the rate you specified.

{{<info>}}
**NOTE:** A Portworx service restart is not required to enable this feature.
{{</info>}}



## Disable RelaxedReclaim

To disable the RelaxedReclaim feature, set the value back to `0`:

```text
pxctl cluster options update --relaxedreclaim-delete-seconds 0
```

When you disable this feature, Portworx will delete any volumes or snapshots that are staged in the RelaxedReclaim queue immediately.

{{<info>}}
**NOTE:** You do not need to restart the Portworx service to disable this feature.
{{</info>}}

## Configure the maximum number of pending operations

If you have too many volumes and too much of a delay time between each deletion operation, your queue could potentially become too long. To prevent this, you can set a number of maximum pending deletion operations using the `relaxedreclaim-max-pending` cluster option. When Portworx hits the maximum pending limit, it will not stage any further volume snapshot deletion operations, instead it will perform those operations immediately. 

```text
pxctl cluster options update --relaxedreclaim-max-pending 3
```

{{<info>}}
**NOTE:** This is a per-node limit.
{{</info>}}

<!-- This limit is NOT in seconds, correct? It's in number of queued operations? -->
