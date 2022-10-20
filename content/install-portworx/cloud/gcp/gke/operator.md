---
title: Install Portworx on Google Kubernetes Engine
linkTitle: Install on GKE
weight: 200
keywords: Install, GKE, Google Kubernetes Engine, k8s, gcloud
description: Install Portworx with Google Kubernetes Engine (GKE).
aliases:
    - /portworx-install-with-kubernetes/cloud/gcp/gke/operator/
    - /portworx-install-with-kubernetes/cloud/gcp/gke/daemonset/
    - /install-portworx/gcp/gke/daemonset/
---

This document provides instructions for installing Portworx with Google Kubernetes Engine (GKE).

## Prerequisites 

Before installing Portworx Enterprise, make sure that you meets the following requirements:

* **Environment requirements**:
    * **Image type**: Only GKE clusters provisioned on [Ubuntu Node Images](https://cloud.google.com/kubernetes-engine/docs/node-images) support Portworx. You must specify the Ubuntu node image when you create clusters.
    * **Resource requirements**: 
        * A GCP GKE cluster that meets the [prerequisites](/install-portworx/prerequisites/). Your [machine-type](https://cloud.google.com/compute/docs/general-purpose-machines) must meet the hardware requirements defined in the [prerequisites](/install-portworx/prerequisites/#installation-prerequisites).
        * For production environments, Portworx recommends 3 Availability Zones (AZs) with one node per zone
        * Portworx recommends you set Max storage nodes per availability zone, Portworx will ensure that many storage nodes exist in the zone
        * Portworx recommends creating a zonal cluster. For instructions on creating a cluster, refer to the [GCP documentation](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster).
* **Permissions**: Portworx requires access to the Google Cloud APIs to provision & manage disks. Make sure that the user or service account creating the GKE cluster has the following roles:
    * Compute Admin
    * Service Account User
    * Kubernetes Engine Cluster Viewer

## Provide permissions to Portworx 

Portworx requires a ClusterRoleBinding for your user to deploy the specs. Create a ClusterRoleBinding using the following `kubectl` command:

```text
kubectl create clusterrolebinding myname-cluster-admin-binding \
    --clusterrole=cluster-admin --user=`gcloud info --format='value(config.account)'`
```

## Install Portworx

{{<info>}}
**NOTE:** Portworx gets its storage capacity from the block storage mounted in the nodes and aggregates the capacity across all the nodes. This way, it creates a **global storage pool**. In this example, Portworx uses Persistent Disks (PD) as that block storage, where Portworx adds PDs automatically as the Kubernetes scales-out and removes PDs as nodes exit the cluster or get replaced.
{{</info>}}

### Generate the specs

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

1. Navigate to <a href="https://central.portworx.com" target="tab">PX-Central</a> and log in, or create an account.

2. Select the Portworx Enterprise option and click **Continue**:

    ![Px-central Product Catalog](/img/pxcentral-install.png)

3. Choose an appropriate license for your requirement and click **Continue**:

    ![Screenshot showing Portworx license selector](/img/pxcentral-license.png)

4. Select your desired Portworx version, select **Built-in** for **ETCD**, and click **Next**.

    ![Screenshot showing Portworx version and ETCD selection](/img/install-shared/basic.png)

5. Select **Cloud**, then select **Google cloud/GKE**. Keep the recommended defaults and click **Next**.

    ![Screenshot showing Portworx version and ETCD selection](/img/gke/select-gke.png)

    {{<info>}}**NOTE**: **Do not add volumes of different types** (standard and ssd) when configuring storage devices during spec generation. This can cause performance issues and errors.{{</info>}}

6. Keep the defaults for **Network** and click **Next**.

    ![Screenshot showing Portworx version and ETCD selection](/img/install-shared/basic.png)

7. In the **Are you running on either of these?** section, select **Google Kubernetes Engine (GKE)**, then click **Finish**.

    ![Screenshot showing Portworx version and ETCD selection](/img/gke/are-you-running.png)

    <br>The spec generator outputs two `kubectl` commands.

8. To install the Operator, run the first command provided by the spec generator. This command will look similar to the following:

    ```
    kubectl apply -f 'https://install.portworx.com/<portworx-version>?comp=pxoperator'
    ```

9. To install your deployment spec, run the second command provided by the spec generator. This command will look similar to the following:

    ```
    kubectl apply -f 'https://install.portworx.com/<portworx-version>?operator=true&mc=false&b=true&kd=type%3Dpd-standard%2Csize%3D150&s=%22type%3Dpd-standard%2Csize%3D150%22&c=px-cluster-c5d0f3c4-8344-46b0-95cd-f0a8fba93736&gke=true&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true'
    ```

{{< content "shared/post-installation.md" >}}
