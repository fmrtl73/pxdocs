---
title: IO throttling
keywords: stork, kubernetes, k8s
description: 
weight: 910
---

<!-- Consider putting this in the CLI section, rather than here. -->

Portworx volumes are created from a common backend pool and share their available IOPS and bandwidth amongst all other provisioned Portworx volumes. The Portworx IO throttling feature provides you with a method for controlling an individual Portworx volume’s IOPS or bandwidth usage of backend pool resources. Some of the advantages of this feature include:

* **Prevents starvation:** Without this feature, one Portworx volume could take over the entire bandwidth available in the backend or the network (for remote attached devices). This can happen due to unintentional jobs scheduled on Portworx volumes in the production cluster (e.g. an `fio` job on a Portworx volume) or an application software bug.
* **Prevents cascading failures:** IO generated from a small number of Portworx volumes could overwhelm the backend bandwidth of one pool or node. This can delay processing on one node, and block other Portworx volumes used on other nodes. In a worst-case scenario, this could cause multiple nodes on the cluster to go offline. IO throttling addresses this issue.
* **Adds IO specific resource management:** When Portworx is working together with Kubernetes, this completes the Kubernetes Resource Management for pods and containers in a significant way by adding an IO specific resource management mechanism to the existing CPU and memory resource management.
* **Persists through volume administration:** This feature adds the IO throttling limit as an intrinsic attribute of Portworx volumes, meaning that the whole Portworx cluster can observe it regardless of normal volume administration operations. For example, the throttling limit is observed after a volume is mounted to a different node.
* **Updates in real time:** Nondisruptive real-time updates to the IO throttling limits are supported, and you can observe throttling effects immediately after updating.


{{<info>}}
**NOTE:** By default, no IO throttling is set for any volumes.
{{</info>}}

## Create volumes

When you create volumes using `pxctl`, you can specify the `--max_bandwidth` and `--max_iops` flags to restrict the storage resources that volume uses:

```text
pxctl volume create --max_bandwidth <ReadBW>,<WriteBW> <volname/volid> 
```
```text
pxctl volume create --max_iops <ReadIOPS>,<WriteIOPS> <volname/volid> 
```

## Update volumes

If you want to add IO throttling to an existing volume, or change its throttling values, you can do so using the `--max_bandwidth` and `--max_iops` flags with the `pxctl volume update` commands:

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