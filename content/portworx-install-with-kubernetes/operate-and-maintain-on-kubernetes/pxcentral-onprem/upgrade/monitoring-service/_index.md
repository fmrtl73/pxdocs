---
title: Upgrade the monitoring service
weight: 3
keywords: Upgrade, PX-Central, On-prem, license, GUI, k8s, monitoring service
description: Learn how to upgrade the monitoring sercice
noicon: true
---

If you've installed the monitoring service using Helm, you can use Helm to upgrade it as well.

## Prerequisites

The monitoring service must already be installed.

## Upgrade

Follow the steps in this section to upgrade the monitoring service component using Helm.

1. Update your Helm repos:

    ```text
    helm repo update
    ```

2. Retrieve all custom values you used during install. Enter the following `helm get values` command to generate a YAML file, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    helm get values --namespace <namespace> px-monitor -o yaml > values.yaml
    ```

3. Delete the post-install hook job. Enter the following `kubectl delete job` command, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    kubectl -n <namespace> delete job pxcentral-monitor-post-install-setup
    ```

4. Remove the `pxcentral-prometheus-operator` deployment. Enter the following `kubectl delete deployment` command, adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    kubectl -n <namespace> delete deployment pxcentral-prometheus-operator
    ```

5. Upgrade the monitoring service component. Run the following `helm upgrade` command, using the `-f` flag to pass the custom `values.yaml` file you generated above and adjusting the value of the `<namespace>` parameter to match your environment:

    ```text
    helm upgrade px-monitor portworx/px-monitor --namespace <namespace>  -f values.yaml
    ```
