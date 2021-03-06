---
title: Create snapshots
weight: 2
hidesections: true
keywords: portworx, container, Kubernetes, storage, Docker, k8s, flexvol, pv, persistent disk, snapshots, stork, clones
description: Learn to take a snapshot of a volume from a Kubernetes persistent volume claim (PVC) and use that snapshot as the volume for a new pod. Try today!
---

This document will show you how to create snapshots of Portworx volumes and how you can clone those snapshots to use them in pods.

{{<info>}}
**Note:** The suggested way to manage snapshots on Kuberenetes is to use STORK. If you are looking to create Portworx snapshots using PVC annotations, you will find [instructions here](/portworx-install-with-kubernetes/storage-operations/create-snapshots/snaps-annotations).
{{</info>}}

## Snapshot types
Using STORK, you can take 2 types of snapshots:

1. [Local](/portworx-install-with-kubernetes/storage-operations/create-snapshots/snaps-local): These are per volume snapshots where the snapshots are stored locally in the current Portworx cluster's storage pools.
2. [Cloud](/portworx-install-with-kubernetes/storage-operations/create-snapshots/snaps-cloud): These snapshots are uploaded to the configured S3-compliant endpoint (e.g AWS S3).

## 3DSnaps
{{<info>}}
**Note:** 3DSnaps are supported in Portworx version 1.4 and above and Stork version 1.2 and above. 3DSnaps are not supported on Kubernetes on DC/OS.
{{</info>}}

3DSnaps is the umbrella term that covers PX-Enterprise's capability to provide app-consistent cluster wide snapshots whether they are local or cloud.

For each of the snapshot types, Portworx supports specifying pre and post rules that are run on the application pods using the volumes. This allows users to quiesce the applications before the snapshot is taken and resume I/O after the snapshot is taken. The commands will be run in pods which are using the PVC being snapshotted.

Read [Configuring 3DSnaps](/portworx-install-with-kubernetes/storage-operations/create-snapshots/snaps-3d) for further details on 3DSnaps.
