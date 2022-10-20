---
title: Create and use local snapshots
weight: 100
linkTitle: Local snapshots
keywords: local snapshots, stork, kubernetes, k8s
description: Learn to take a snapshot of a volume from a Kubernetes persistent volume claim (PVC) and use that snapshot as the volume for a new pod.
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-snapshots/on-demand/snaps-local/
---
This document will show you how to create snapshots of Portworx volumes and how you can clone those snapshots to use them in pods.

{{<info>}}
The suggested way to manage snapshots on Kuberenetes is to use Stork. If you are looking to create Portworx snapshots using PVC annotations, you will find [instructions here](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-annotations).
{{</info>}}

## Prerequisites

**Install Stork**

This requires that you already have [Stork](/operations/operate-kubernetes/storage-operations/stork) installed and running on your
Kubernetes cluster. If you fetched the Portworx specs from the Portworx spec generator in [PX-Central](https://central.portworx.com) and used the default options, Stork is already installed.

## Create snapshots

With local snapshots, you can either snapshot individual PVCs one by one or snapshot a group of PVCs by using a label selector.

{{<homelist series="k8s-local-snap">}}

## Restore snapshots

Once you've created a snapshot, you can restore it to a new PVC or the original PVC.

### Restore a local snapshot to a new PVC

{{< content "shared/portworx-install-with-kubernetes-storage-operations-create-snapshots-k8s-restore-pvc-from-snap.md" >}}

### Restore a local snapshot to the original PVC

{{< content "shared/portworx-install-with-kubernetes-storage-operations-create-snapshots-k8s-in-place-restore-pvc-from-snap.md" >}}
