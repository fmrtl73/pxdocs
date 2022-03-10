---
title: Operator OpenShift upgrade path
keywords: Operator, OpenShift, OCP, upgrade
description: Review which OpenShift versions you should upgrade to if you're running Portworx using the Operator.
weight: 6
series: kb
---

## Portworx 2.8.0

This upgrade path was tested with Portworx 2.8.0 and Operator 1.6.0.

{{<info>}}
**NOTE:** If you are not on the same OCP version as in the tested upgrade example shown below, please upgrade to the next available version that is used in the upgrade path and continue as shown in the table below.
{{</info>}}

| **Channel version** | **OpenShift version** | **K8s version** |
|---------------------|-----------------------|-----------------|
| stable-4.6          | 4.6.48                | v1.19.14        |
| stable-4.7          | 4.7.39                | v1.20.11        |
| stable-4.8          | 4.8.23                | v1.21.6         |
| stable-4.9          | 4.9.10                | v1.22.3         |
