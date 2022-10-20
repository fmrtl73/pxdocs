---
title: Enable pay-as-you-go billing on an air-gapped cluster
linkTitle: Air-gapped pay-as-you-go billing
keywords: pay-as-you-go, PAYG, air-gapped
description: Learn how you can enable pay-as-you-go billing for Portworx on an air-gapped cluster
weight: 400
aliases:
    - /reference/licensing/portworx-enterprise/pay-as-you-go-air-gapped/
---

You can enable pay-as-you-go billing for an air-gapped cluster with no outbound connectivity by acquiring a pay-as-you-go account key from Portworx.

{{<info>}}
**Note:** Once you enable a pay-as-you-go license for an air-gapped cluster with no outbound connectivity, you currently can't switch to any other license.
{{</info>}}

## Prerequisites

* An air-gapped Portwox Kubernetes cluster
* An air-gapped SaaS key from Portworx

## Enable pay-as-you-go billing on an air-gapped cluster

1. Activate the pay-as-you-go license key that you received from Portworx.

    ```text
    pxctl license activate saas --key <pay-as-you-go-key>
    ```

2. Verify if the license is activated.

    ```text
    /opt/pwx/bin/pxctl  status
    ```
    ```output
    Status: PX is operational
    Telemetry: Disabled or Unhealthy
    Metering: Healthy
    License: PX-Enterprise Usage-Based-Airgapped
    Node ID: 9e743afc-5178-4915-a926-0d74f9801263
    ```
3. Download your consumption report.

    ```text
    pxctl service usage-report –download \
    –month <month number to download report for> \
    —year <year for which to download report>
    ```
 This command creates a tarball on your system, which you can email to the support team for billing.

