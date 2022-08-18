---
title: Install Portworx on Kubernetes using Rancher
linkTitle: Install Portworx
keywords: Install, Rancher 2.x, Helm
description: Install Portworx on Kubernetes using Rancher 2.x with a public catalog (Helm Chart)
weight: 600
aliases: 
    - /install-with-other/rancher/rancher-1.x
---

This guide provides instructions for installing Portworx on Kubernetes using Rancher with a Helm chart.

{{<info>}}
**NOTE:** Currently, Portworx does not support the `RancherOS` distro.
{{</info>}}

## Prerequisites

* You must have a Kubernetes cluster imported into Rancher
* Your cluster must meet the [requirements](/install-portworx/prerequisites/) for installing a Portworx cluster

## Install Portworx on Kubernetes using Rancher

Perform the following steps to install Portworx from your Rancher UI:

1. From your **Cluster Dashboard**, click **App** in the left pane to open the **Charts** page:

    ![Rancher Dashboard](/img/rancher-dasboard.png)

2. Search for the Portworx catalog and select the **Portworx** card on the **Charts** page:

    ![Search Portworx](/img/rancher-search-px.png)

3. Choose the latest version from the **Chart Versions** list and then click the **Install** button in the upper-right corner to start the Helm chart form:

    ![Install Portworx](/img/rancher-install-px.png)

4. Choose the **kube-system** namespace from the drop-down list and click **Next**:

    ![Portworx Namespace](/img/rancher-namespace.png)

5. Populate the various sections of the **Edit Options** of the Helm chart form:

     * In the **Key value store parameters** section, choose **Built-in** option for the **Select ETCD** filed.

     * In the **Storage Parameters** section, specify whether your cluster is located on-prem or on a cloud provider, and choose your disk configuration:
     
        ![Portworx-parameters](/img/rancher-portworx-parameters.png)
    
    * Do not change the Portworx version in the **Portworx version to be deployed** field of **Advanced parameters**.
        ![Advance parameters](/img/rancher-advance-parameter.png)

6.  Click the **Install** button in the bottom-right corner to deploy Portworx.

    Once the installation is completed, the Portworx status will be shown as Deployed.

## Post-Install

Once you have a running Portworx installation, below sections are useful.

{{<homelist series2="k8s-postinstall">}}

