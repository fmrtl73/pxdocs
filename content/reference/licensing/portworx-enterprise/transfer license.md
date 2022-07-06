---
title: Transfer licenses
keywords: transfer license, enterprise license
description: Learn how you can transfer license between two Portworx clusters.
weight: 500
---

You can transfer a valid enterprise license between two Portworx clusters. Both Portworx clusters need to be operational at the time of license transfer.
The source cluster must have a valid {{< pxEnterprise >}} license, while the destination cluster can have either a valid or expired Trial license or expired {{< pxEnterprise >}} license. 

License transfer command requires `clusterUUID` from the source cluster, (available via `pxctl cluster list` command) and remote Portworx cluster node IP.


```text
pxctl license transfer -h
```

```output
NAME:
   pxctl license transfer - Transfer license to remote PX cluster

USAGE:
   pxctl license transfer <clusterUUID> <remoteIP>

DESCRIPTION:
   Command swaps licenses between the two Portworx clusters.

   Note that both Portworx clusters need to be operational at the time this
   command is ran.
   The source cluster must have a valid {{< pxEnterprise >}} license, while the
   destination cluster can have either a valid or expired Trial license or expired {{< pxEnterprise >}} license.

EXAMPLE:
   pxctl license transfer f91531d9-bf65-46f5-9619-eb99128e3270 10.0.15.201
```
{{<info>}}
**NOTE:**

 * The license transfer happens directly between the Portworx clusters, so at least one node from source cluster must have a network connectivity to a node in the target Portworx cluster.
 * After the successful license transfer, the two Portworx clusters swap identities and licenses (for example, Portworx cluster A will have Trial license originally from Portworx cluster B, 
 while the cluster B will have the {{< pxEnterprise >}} license originally from cluster A)
{{</info>}}

For information on purchase, upgrades and support, [contact support](/support/)