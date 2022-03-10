---
title: Operate Portworx on VMware Tanzu
description: Learn how to operate Portworx cluster on VMware Tanzu.
keywords: VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: Operate
weight: 4
noicon: true
---

A Portworx cluster on Tanzu can contain a mixture of storage and storageless nodes. When scaling down a Tanzu cluster, one must be careful on not scale down the cluster size lower than the total number of storage nodes in the Portworx cluster.

You can find out the number of storage nodes by running `pxctl status` on any PX node. This will list all nodes in the cluster and mention which ones are storage nodes.

To control the number of storage nodes in the cluster, use the `spec.cloudStorage.maxStorageNodesPerZone` or `spec.cloudStorage.maxStorageNodes` configuration in [`StorageCluster`](/reference/crd/storage-cluster/).
