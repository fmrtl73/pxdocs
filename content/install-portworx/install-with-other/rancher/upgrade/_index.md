---
title: Upgrade Portworx on Kubernetes using Rancher 
linkTitle: Upgrade Portworx
keywords: upgrade, Portworx, Rancher, Helm chart
description: Upgrade Portworx on Kubernetes using Rancher with a public catalog (Helm Chart)
aliases: 
    - /install-with-other/rancher/rancher-2.x/upgrade

---

## Purpose

This page provides instructions for upgrading Portworx on Kubernetes using Rancher with a Helm chart available from the public catalog.

## Upgrade Portworx

Perform the following steps to upgrade Portworx from your Rancher UI:

1. Click **Apps** and then click **Installed Apps** in the left pane. 

2. Select **All Namespaces** from the dropdown on the top toolbar. You can see Portworx under kube-system namespace:

    ![Rancher apps page](/img/rancher-upgarde-app.png)

2. The latest Portworx version will be highlighted under **Upgradable**. Click that number and then click the **Next** button in the bottom-right corner:

    ![Rancher upgrade](/img/rancher-upgrade-version-latest.png)

3.  Click **Upgrade**:

    ![Rancher Upgrade parameters](/img/rancher-upgarde-parameter.png)

    Rancher will start upgrading Portworx. After the upgrade is complete the status of Portworx will change to Deployed.