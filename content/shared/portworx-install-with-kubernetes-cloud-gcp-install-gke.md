---
title: Shared content for GCP
hidden: true
description: Setup a production ready Portworx cluster Google Cloud Platform (GCP).
keywords: Install, GKE, Google Kubernetes Engine, k8s, Google Cloud, gcloud
---

{{<info>}}
_Portworx_ gets its storage capacity from the block storage mounted in the nodes and aggregates the capacity across all the nodes. This way, it creates a **global storage pool**. In our example, _Portworx_ uses Persistent Disks (PD) as that block storage, where _Portworx_ adds PDs automatically as the Kubernetes scales-out and removes PDs as nodes exit the cluster or get replaced.
{{</info>}}

{{< content "shared/portworx-install-with-kubernetes-shared-1-generate-the-spec-footer.md" >}}

{{< content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" >}}
