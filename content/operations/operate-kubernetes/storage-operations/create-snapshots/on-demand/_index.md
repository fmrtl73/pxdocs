---
title: On-demand snapshots
weight: 200
hidesections: true
linkTitle: On-demand snapshots
keywords: on-demand snapshots, local snapshots, cloud snapshots, stork, kubernetes, k8s
description: Learn how to create on-demand consistent snapshots/backups and restore them.
series: k8s-storage-snapshots
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-snapshots/on-demand/snaps-group/
    - /portworx-install-with-kubernetes/storage-operations/create-snapshots/on-demand/snaps-group-cloud/
    - /portworx-install-with-kubernetes/storage-operations/create-snapshots/on-demand/snaps-group-legacy/
---

This document will show you how to create snapshots of Portworx volumes and how you can clone those snapshots to PVCs and use them with your applications.

{{<info>}}
The suggested way to manage snapshots on Kubernetes is to use Stork. If you are looking to create Portworx snapshots using PVC annotations, you will find [instructions here](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-annotations).
{{</info>}}

## Snapshot types

Using Stork, you can take 2 types of snapshots:

1. [Local](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-local): These are per volume snapshots where the snapshots are stored locally in the current Portworx cluster's storage pools.
2. [Cloud](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-cloud): These snapshots are uploaded to the configured S3-compliant endpoint (e.g AWS S3).
