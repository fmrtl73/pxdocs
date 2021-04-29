---
title: Upgrade PX-Central on-premises
description: Upgrade your version of PX-Central on-premises
keywords: Upgrade, PX-Central, On-prem, license, GUI, k8s
weight: 1
noicon: true
hideSections: true
---

If you've installed PX-Central using Helm, you can use Helm to upgrade it as well.

## Prerequisites

You must have PX-Central with a Helm-based install.
## Upgrade

Follow the steps in this section to upgrade PX-Central using Helm.

1. Update your Helm repos:

    ```text
    helm repo update
    ```

2. Retrieve all custom values you used during install. Enter the following `helm get values` command to generate a YAML file, adjusting the values of the `<namespace>` and `<release-name>` parameters to match your environment:

    ```text
    helm get values --namespace <docs-namespace> <release-name> -o yaml > values.yaml
    ```

    ```
    oidc:
        centralOIDC:
            defaultPassword: examplePassword
            defaultUsername: exampleUser
    operatorToChartUpgrade: true
    persistentStorage:
        enabled: true
        storageClassName: px-sc
    pxbackup:
        orgName: exampleOrg
    pxcentralDBPassword: exampleDbPassword
    ```

    Note the following about this example output:

    * The `persistentStorage.storageClassName` field displays the name of your storage class (`px-sc`).
    * The `persistentStorage.enabled: true` field indicates that persistent storage is enabled.
    * The `pxbackup.orgName` field displays the name of your organization (`my-organization`)

3. Delete the post install hook job:

    ```text
    kubectl delete job pxcentral-post-install-hook --namespace <namespace>
    ```

4. Run the `helm upgrade` command, using the `-f` flag to pass the custom `values.yaml` file you generated above and replacing `<namespace>` with your namespace:

    ```text
    helm upgrade px-backup portworx/px-backup --namespace <namespace>  -f values.yaml
    ```