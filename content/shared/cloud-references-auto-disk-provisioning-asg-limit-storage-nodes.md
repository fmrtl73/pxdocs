---
hidden: true
---

Portworx allows you to create a heterogenous cluster where some of the nodes are storage nodes and rest of them are storageless.

You can specify the number of storage nodes in your cluster by setting the `max_storage_nodes_per_zone` input argument.
This instructs Portworx to limit the number of storage nodes in one zone to the value specified in max_storage_nodes_per_zone argument. The total number of storage nodes in your cluster will be:

_Total Storage Nodes = (Num of Zones) * max_storage_nodes_per_zone_

While planning capacity for your auto scaling cluster make sure the minimum size of your cluster is equal to the total number of storage nodes in Portworx. This ensures that when you scale up your cluster, only storage less nodes will be added. While when you scale down the cluster, it will scale to the minimum size which ensures that all Portworx storage nodes are online and available.

{{<info>}}You can always ignore the **max_storage_nodes_per_zone** argument. When you scale up the cluster, the new nodes will also be storage nodes but while scaling down you will lose storage nodes causing Portworx to lose quorum. {{</info>}}
