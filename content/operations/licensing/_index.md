---
title: Portworx licenses
keywords: Licensing, Portworx Enterprise, px-developer, requirements, Portworx features, trial period, upgrade Portworx, activate license, transfer license
description: Find out more about the Portworx licensing model.
weight: 400
series: operations
hidesections: true
aliases: 
    - /reference/knowledge-base/licensing/licensing-operations/
    - /reference/licensing/
---

## Introduction

This topic explains the various Portworx license types, and how you can use them.

{{<info>}}
**Note:** If you have already set up your cluster using any of the paid or free Portworx licenses, you can switch to pay-as-you-go (PAYG) billing by acquiring a PAYG account key from the Pure Storage support team. For more information about switching to PAYG, refer to the [pay-as-you-go](/operations/licensing/portworx-enterprise/pay-as-you-go) topic.
{{</info>}}

Portworx products support the following license types:

|      License type      |  Description
|:-----------------------|:-------------------------------------------------------------------------------------------------------------------------------
| Portworx Essentials           | Free Portworx license with limited functionality, suitable to run small production or proof-of-concept workloads.
| Trial                  | Default license installed with 30 days trial period for full Portworx Enterprise functionality.
| {{< pxEnterprise >}} VM       | Enterprise license, suitable for Virtual Machine (VM) installs on-prem and in cloud
| {{< pxEnterprise >}} Metal    | Enterprise license, suitable for installs on any bare metal hardware


Depending on the type of container you are installing, a different license will be automatically activated. For example, the Portworx Enterprise automatically activates the Trial license (limited to 30 days), which you can upgrade to a {{< pxEnterprise >}} license at any time.

## Licensed features

In the following table, you can see the overview of features that are controlled via licensing.

|       Description            |  Type  | Details
|:-----------------------------|:------:|:------------------------------------------------------------------------------------
| Number of nodes maximum      | number | Defines the maximum number of nodes in a cluster
| Number of volumes maximum    | number | Defines max number of volumes on a single node
| Volume capacity [TB] maximum | number | Defines max size of a single volume
| Storage aggregation          | yes/no | Defines if volumes may be aggregated across multiple nodes
| Shared volumes               | yes/no | Defines if volumes may be shared w/ other nodes
| Volume sets                  | yes/no | Defines if volumes may be scaled
| BYOK data encryption         | yes/no | Defines if volumes may be encrypted
| Resize volumes on demand     | yes/no | Defines if volumes can be resized
| Snapshot to object store     | yes/no | Defines if volumes may be snapshotted to Amazon S3, MS Azure and Google storage
| Cluster level migration      | yes/no | Defines if applications and data (using K8s namespace) can be migrated between paired clusters
| Disaster Recovery (PX-DR)[Add-on]    | yes/no | Enables synchronous and asynchronous DR features (requires 2.1 or later, needs additional license)
| Virtual machine hosts        | yes/no | Software may be deployed on VMs (including Amazon EC2, OpenStack Nova, etc...)
| Bare-metal hosts             | yes/no | Software may be deployed on commodity hardware


## Trial License

The trial license activates automatically when the [Portworx Enterprise license](/operations/licensing/portworx-enterprise) is installed.
The trial license provides the full product functionality for 30 days.

```
DESCRIPTION                  ENABLEMENT  ADDITIONAL INFO
Number of nodes maximum         1000
Number of volumes maximum       1024
Volume capacity [TB] maximum      40
Storage aggregation              yes
Shared volumes                   yes
Volume sets                      yes
BYOK data encryption             yes
Resize volumes on demand         yes
Snapshot to object store         yes
Bare-metal hosts                 yes
Virtual machine hosts            yes
Product SKU                     Trial    expires in 6 days, 20:40
```


#### Trial license expiration

After the trial period expires, you cannot create new volumes or volume snapshots.
You can restore normal functionality by purchasing and installing a {{< pxEnterprise >}} license.  