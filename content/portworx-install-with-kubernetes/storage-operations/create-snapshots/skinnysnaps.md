---
title: SkinnySnaps
weight: 6
keywords: snapshots, on-demand, scheduled, consistent snapshots, kubernetes, k8s, skinnySnaps
description: Learn what skinnySnaps are and how to use them.
---

Portworx SkinnySnaps provide you with a mechanism for controlling how your cluster takes and stores volume snapshots. This allows you to optimize performance by configuring how many snapshot replicas your cluster stores independently from the number of volume replicas you have. 

### Default snapshot behavior

Regular snapshots take volume snapshots for each volume replica up to the retain limit. 

Consider the following example:

* You're using a 3 node cluster
* You have a repl3 volume on that cluster with a retain of 3

This means that:

* You have one volume replica on each node
* each volume replica has all (3) accompanying snapshots

![](/img/skinnySnapsRegular.png)

When Portworx reaches the limit of the snapshot retain policy, it replaces the oldest snapshot with the newest:

![](/img/skinnySnapsRegularAdd.png)

### Default snapshot limitations

Default snapshots guarantee that all snapshots retain the same replication factor as their parent volumes. If a node with a highly available volume goes down, your snapshots will remain unaffected. 

However, these snapshot operations can generate a large amount of concurrent IOPS for both your cluster drives and filesystem on any node that has a volume replca. This can impact latency, particularly because a snapshot operation triggers these IOPS all-at-once. 

### SkinnySnap behavior

Using SkinnySnaps, you can define a snapshot replication factor independently from the volume replication factor. 

Consider the following example:

* You're using a 3 node cluster
* You have a repl3 volume on that cluster with a retain of 3
* You're using a snapshot replication factor of 1

This means that:

* You have one volume replica on each node
* Not all volume replicas have all accompanying snapshots

![](/img/skinnySnaps.png)


When you take a snapshot, Portworx takes only 1 replica. Because the retain policy is still 3, Portworx replaces the oldest snapshot with the newest:

![](/img/skinnySnapsAdd.png)

### SkinnySnap limitations

Under this method, not all of your snapshots reside with all of your volume replicas, so you lose HA factor. If a node goes down, even if another node contains a volume replica, you may still lose some snapshots:

![](/img/skinnySnapsNodeFail.png)

To mitigate this, while still improving performance over the default snapshot method, use a higher snapshot replication factor.

{{<info>}}
**NOTE:** You cannot use skinnySnaps to increase the snapshot replication factor beyond the number of volume replicas. In other words, if you have a volume with a replication factor of 2, you cannot have a snapshot replication factor of 3.
{{</info>}}

### Replica selection

Portworx guarantees that if a node goes down, it'll take a minimum number of snapshots down with it by distributing snapshot replicas over as many nodes as possible. How you configure the snapshot replication factor and how you configure the determine the tradeoff you make between performance improvement and snapshot durability.

Consider the following scenarios for a 3 node cluster: 

* You need a minimum snapshot replication factor of 2 to ensure that if a node goes down, you'll still have at least 1 replica remaining.
* If you have a snapshot replication factor of 3, you will see no performance improvements over traditional snapshots.
* A snapshot replication factor of 1 is the most performant, but you will lose a snapshot if a node fails. 

<!-- #### BTRFS optimize
 -->


<!-- 
#### Load balance

accross all nodes, look at # replicas each node has, try and distribute replicas across all nodes so that load is similar across the cluster.

Specifically, when a snapshot is taken, Portworx looks at all the nodes where a snapshot can be taken, the total # of replicas for volumes it has, and tries to divide the workload.  
-->

## Use SkinnySnaps

### Enable SkinnySnaps

By default, the skinnySnaps feature is disabled. You must enable it as a cluster option. 

Enter the following `pxctl` command:

```text
pxctl cluster options update --skinnysnap on
```
```output
SkinnySnap causes snapshot to have less replication factor than the parent volume. Do you still want to enable SkinnySnap? (Y/N): Y
Successfully updated cluster-wide options
```

<!-- This doesn't seem to work if you're using kubectl exec. You can't confirm the operation:

    kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl cluster options update --skinnysnap on
    Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
    SkinnySnap causes snapshot to have less replication factor than the parent volume. Do you still want to enable SkinnySnap? (Y/N): Not updating cluster options

 -->


### Configure SkinnySnaps

Once you've enabled SkinnySnaps for your cluster, you can configure them using cluster options:

```text
pxctl cluster options update --skinnysnap-num-repls 2
```
```output
Successfully updated cluster-wide options
```


### Restore SkinnySnaps

Replication for SkinnySnaps differ from the parent volumes. Use one of the following methods to restore these kinds of snapshots:

#### Perform an in-place restore

1. [Reduce the parent volume replication factor](/reference/cli/updating-volumes/#decreasing-the-replication-factor) to match with the replication of SkinnySnap and the replica location.
2. [Perform an in-place restore](/reference/cli/snapshots/#restoring-snapshots).
3. [Increase the replication factor](/reference/cli/updating-volumes/#increase-the-replication-factor) to restore high availability to the parent volume.

#### Clone the SkinnySnap

1. Clone the SkinnySnap and increase the replication factor to a desired value. 
2. Use a [pre-provisioned volume](/portworx-install-with-kubernetes/storage-operations/create-pvcs/using-preprovisioned-volumes/) to point the application to this new clone. 