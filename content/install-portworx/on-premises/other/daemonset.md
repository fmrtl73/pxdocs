---
title: Install Portworx on-prem using the DaemonSet
linkTitle: Install using the DaemonSet
weight: 300
keywords: Install, on-premises, kubernetes, k8s, air gapped
description: How to install Portworx with Kubernetes
noicon: true
aliases:
    - /portworx-install-with-kubernetes/on-premise/other/daemonset/
---

This topic provides instructions for installing Portworx with Kubernetes on-prem using the DaemonSet.

## Install

{{<info>}}**Airgapped clusters**: If your nodes are airgapped and don't have access to common internet registries, first follow [Airgapped clusters](/install-portworx/on-premises/airgapped) to fetch Portworx images.{{</info>}}

{{< content "shared/portworx-install-with-kubernetes-shared-1-generate-the-spec-footer.md" >}}

{{< content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" >}}

{{< content "shared/portworx-install-with-kubernetes-post-install.md" >}}

## Related topics

* Refer to the [Advanced installation parameters](/operations/operate-kubernetes/other-operations/advanced-installation-parameters/) page for details about how to specify advanced installation parameters.
