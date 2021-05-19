---
title: Installation
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

Starting with the Portworx Operator 1.5 and higher, CSI is installed by default unless explicitly disabled. Due to the nature of CSI and its dependencies, it is highly recommend to use the Portworx Operator instead of the spec generator for installation.

The Portworx Operator easily manages all CSI components based on your Kubernetes and Portworx versions. This makes upgrading Kubernetes and Portworx versions far easier.

## Portworx Operator 1.4 and earlier

Older operator versions do not enable CSI by default. It is highly recommended to upgrade to Portworx Operator 1.5 or later.

To enable CSI on older Portworx Operator versions, you may set the following CSI feature gate:

```

```

{{<info>}}**Openshift users**:
You must add the `px-csi-account` service account to the privileged security context.

```text
oc adm policy add-scc-to-user privileged system:serviceaccount:kube-system:px-csi-account
```
{{</info>}}

## Contribute

Portworx, Inc. welcomes contributions to its CSI implementation, which is open-source and repository is at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).