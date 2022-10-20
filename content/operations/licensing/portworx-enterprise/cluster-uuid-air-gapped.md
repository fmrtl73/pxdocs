---
title: Activate license on an air-gapped cluster using Cluster UUID
linkTitle: Enable using Cluster UUID
description: Learn about the Portworx Enterprise 
keywords: enterprise license, license sharing, transfer license, air-gapped license, Saas Key, Licensing, Portworx Enterprise, upgrade Portworx, activate license
weight: 200
aliases:
    - /reference/licensing/portworx-enterprise/cluster-uuid-air-gapped/
---
1. Get your Cluster UUID and contact Portworx.
The `Cluster UUID` information is available via `pxctl cluster list` command.

    ```text
    pxctl cluster list
    ```

    ```output
    Cluster ID: MY_FAVORITE_PX_CLUSTER
    Cluster UUID: f987ad4b-987c-4e7e-a8bd-788c89cc40f1
    Status: OK [...]
    ```

2. Upon supplying the Cluster UUID, you receive your license file.
Upload this file on one of the Portworx nodes, and activate the license by running the following command:

    ```text
    pxctl license add license_file.bin
    ```

{{<info>}}
**Note:**

 * The license installation is a non-obtrusive process, which does not interfere with the data stored on the Portworx volumes, nor interrupts the active IO operations.

 * If you’re running Portworx on an air-gapped cluster that has no outbound connectivity, refer to [this page](/operations/licensing/portworx-enterprise/pay-as-you-go-air-gapped) for enabling pay-as-go billing.

 * Once you enable a pay-as-you-go license on an air-gapped cluster with no outbound connectivity, you currently can't switch to any other license.
{{</info>}}