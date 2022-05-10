---
title: Upgrade Portworx or Kubernetes on an air-gapped cluster
linkTitle: Upgrade Portworx or Kubernetes on an air-gapped cluster
weight: 2
logo: /logos/other.png
keywords: Install, on-premises, kubernetes, k8s, air gapped
description: How to upgrade Portworx or Kubernetes version in an air-gapped cluster
noicon: true
---

During installation, Portworx fetches required images and packages from the internet. If you're installing Portworx onto an air-gapped cluster, you'll need to perform extra steps to make these resources available.

To upgrade your Portworx or Kubernetes installation to a newer version, or to upgrade any component that updates your Kubernetes version, you must repeat all of the steps you performed for the initial installation. New versions might require new component images, so you must re-run the steps to pre-load those images.

{{<info>}}
**WARNING:** If you do not pre-load images before upgrading, your pods can enter crash loops and cause service disruptions.
{{</info>}}

{{< content "shared/airgap-install-procedure.md" >}}