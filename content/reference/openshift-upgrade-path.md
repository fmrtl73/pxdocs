---
title: Operator OpenShift upgrade path
keywords: Operator, OpenShift, OCP, upgrade
description: Review which OpenShift versions you should upgrade to if you're running Portworx using the Operator.
weight: 900
series: reference
aliases:
  - /reference/knowledge-base/openshift-upgrade-path/
---

## Portworx 2.10.0

This upgrade path was tested with Portworx 2.10.0 and Operator 1.8.0.

{{<info>}}
**NOTE:** If you are not on the same OCP version as in the tested upgrade example shown below, please upgrade to the next available version that is used in the upgrade path and continue as shown in the table below.
{{</info>}}

| **Channel version** | **OpenShift version** | **Kubernetes version**  |
|---------------------|-----------------------|-------------------------|
| stable-4.6          | 4.6.56                | v1.19.16+6175753        |
| stable-4.7          | 4.7.45                | v1.20.14+0d60930        |
| stable-4.8          | 4.8.35                | v1.21.8+ee73ea2         |
| force 4.9[^1]       | 4.9.24                | v1.22.5+5c84e52         |
| fast-4.10           | 4.10.6                | v1.23.5+b0357ed         |

[^1]: The upgrade hop to 4.9 was not publicly available via `stable`, `fast`, or `candidate` channels. `--force --allow-explicit-upgrade --allow-upgrade-with-warnings` was used to bypass this blocker for testing.
