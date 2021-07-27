---
title: Enable CSI
keywords: csi, install, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 1
---


## Prerequisites

Before you install and use Portworx with CSI, ensure you meet the prerequisistes:

* Openshift users must use Openshift 4.1 or higher
* Kubernetes users must use 1.13.12 or higher
* Certain features may require newer versions of Kubernetes or Portworx

## Overview

The Portworx CSI Driver requires additional components called `CSI Sidecars` in order to function properly. To make sure these components and their dependencies are installed on your cluster, follow the instructions below. 

## Portworx CSI installed by default

Starting with the Portworx [Operator 1.5 and higher](/reference/crd/storage-cluster/), CSI is installed by default unless explicitly disabled. Due to the nature of CSI and its dependencies, it is highly recommend to use the Portworx Operator instead of the spec generator for installation.

The Portworx Operator easily manages all CSI components based on your Kubernetes and Portworx versions. This makes upgrading Kubernetes and Portworx versions far easier.

{{<info>}}**Note**:
Explicitly disabling CSI will also prevent the Pure Storage FlashBade DirectAccess mode from working.
{{</info>}}

## Portworx Operator 1.4 and earlier

Older operator versions do not enable CSI by default. It is highly recommended to upgrade to Portworx Operator 1.5 or later.

To enable CSI on older Portworx Operator versions, you may set the following CSI feature gate:

```
spec:
  featureGates:
    CSI: "true"
```

## Openshift installation:

You must add the `px-csi-account` service account to the privileged security context.

```text
oc adm policy add-scc-to-user privileged system:serviceaccount:kube-system:px-csi-account
```


{{<info>}}**Note**:
If you experience the following error:
"errorUnable to update cluster as v1alpha1 version of ... Remove these CRDs to allow the upgrade to proceed", 
follow [this solution from RedHat](https://access.redhat.com/solutions/5372561).
{{</info>}}


## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
