---
title: Upgrade the license server component
weight: 2
keywords: Upgrade, PX-Central, On-prem, license, GUI, k8s, license server
description: Learn how to upgrade the license server component
noicon: true
---

If you've installed the license server component using Helm, you can use Helm to upgrade it as well.

## Prerequisites

The license server component must already be installed.

## Upgrade

Follow the steps in this section to upgrade the license server component using Helm.

1. Update your Helm repos:

    ```text
    helm repo update
    ```

2. Retrieve all custom values you used during install. Enter the following `helm get values` command to generate a YAML file, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    helm get values --namespace <namespace> px-license-server -o yaml > values.yaml
    ```

3. Delete the post-install hook job. Enter the following `kubectl delete job` command, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    kubectl delete job pxcentral-license-ha-setup -n <namespace>
    ```

4. Remove the `pxcentral-license-server` deployment. Enter the following `kubectl delete deployment` command, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    kubectl delete deployment pxcentral-license-server -n <namespace>
    ```

5. Upgrade the license server component. Run the following `helm upgrade` command, using the `-f` flag to pass the custom `values.yaml` file you generated above and adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    helm upgrade px-license-server portworx/px-license-server --namespace <namespace> -f values.yaml
    ```
