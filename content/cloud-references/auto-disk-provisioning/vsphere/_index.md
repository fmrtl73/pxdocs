---
title: Disk Provisioning on VMware vSphere
description: Learn to scale a Portworx cluster up or down on VMware vSphere with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, vSphere ASG, Kubernetes, k8s
linkTitle: VMware vSphere
weight: 3
noicon: true
---

This guide explains how the Portworx Dynamic Disk Provisioning feature works within Kubernetes on VMware and the requirements for it.

{{<info>}}Installation steps below are only supported if you are running with Kubernetes.{{</info>}}

## Architecture

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-shared-arch.md" >}}


## Limiting storage nodes

{{< content "shared/cloud-references-auto-disk-provisioning-asg-limit-storage-nodes.md" >}}

{{< content "shared/cloud-references-auto-disk-provisioning-asg-examples-vsphere.md" >}}

## Availability across failure domains

Since Portworx is a storage overlay that automatically replicates your data, {{<companyName>}} recommends using multiple availability zones when creating your VMware vSphere based cluster. Portworx automatically detects regions and zones that are populated using known Kubernetes node labels. You can also label nodes with custom labels to inform Portworx about region, zones and racks. Refer to the [Cluster Topology awareness](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/cluster-topology/) page for more details.

## Installation

{{<info>}}
**NOTE:**

- Run these steps from a machine which has `kubectl` access to your cluster.
- If you provide a vSphere datastore cluster as an input, Portworx uses a DRS recommendation API that returns the correct datastore to use from the selected datastore cluster.
- vSphere Storage DRS must be enabled and must be in `manual` mode for storage clusters.
{{</info>}}

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-px-install.md" >}}
