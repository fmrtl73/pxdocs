---
title: "Install Portworx on Kubernetes via Helm"
linkTitle: Helm
keywords: install, portworx, kubernetes, k8s, helm, uninstall
description: "Find out how to install Portworx on Kubernetes via the Portworx Helm chart"
weight: 200
hidden: true
---

The Portworx Helm chart deploys Portworx and [Stork](https://docs.portworx.com/scheduler/kubernetes/stork.html) in your Kubernetes cluster.

## Prerequisites

* Helm has been installed on the client machine from where you would install the [chart](https://docs.helm.sh/using_helm/#installing-helm)
* [Portworx pre-requisites](/install-portworx/prerequisites/#installation-prerequisites)

## Install

To install Portworx via the chart with the release name `my-release` run the following commands:

1. Clone the Portworx Helm chart repository.

    ```text
    git clone https://github.com/portworx/helm.git
    ```
2. Specify the values for the options in the [values.yaml](https://github.com/portworx/helm/blob/master/charts/portworx/values.yaml) file as per your requirement.

3. Install the chart using the following command:

    ```text
    helm install my-release ./helm/charts/portworx/ --debug
    ```

## Uninstall

Follow the steps below to wipe your entire Portworx installation.

1. Add the following to the [values.yaml](https://github.com/portworx/helm/blob/master/charts/portworx/values.yaml) file if the `deleteStrategy` parameter was not set during installation: 

    ```text
    deleteStrategy:                          
        type: UninstallAndWipe
    ```
2. Run the following command to use the updated *values.yaml* file:
    
    ```text
    helm upgrade my-release ./helm/charts/portworx/ --debug
    ```

     {{<info>}}**NOTE:** Skip step 1 and step 2 if the `deleteStrategy` parameter was set during installation.{{</info>}}

3. Delete the Helm release by running the following command:

    ```text
    helm delete my-release --debug
    ```

## Post-Install

Once you have a running Portworx installation, below sections are useful.

{{<homelist series2="k8s-postinstall">}}

## Troubleshooting helm installation failures

Refer to the common troubleshooting instructions for Portworx deployments via Helm [Troubleshooting portworx installation](https://github.com/portworx/helm/tree/master/charts/portworx#basic-troubleshooting)
