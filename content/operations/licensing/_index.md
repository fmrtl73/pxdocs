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
| Portworx CSI for FA/FB | Free Portworx license with a limited feature set. It provides essential container storage capabilities for Portworx on FlashArray and FlashBlade and is supported only for DirectAccess. |
| Trial                  | Same as the Portworx Enterprise license. It is a default license installed with 30 days trial period for full Portworx Enterprise functionality |
| {{< pxEnterprise >}} VM       | Enterprise license, suitable for Virtual Machine (VM) installs on-prem and in cloud
| {{< pxEnterprise >}} Metal    | Enterprise license, suitable for installs on any bare metal hardware |


Depending on the type of container you are installing, a different license will be automatically activated. For example, the Portworx Enterprise automatically activates the Trial license (limited to 30 days), which you can upgrade to a {{< pxEnterprise >}} license at any time.

## Licensed features

In the following table, you can see the overview of features that are controlled via licensing.

| License feature                                        | Description                                                                                                                                                                 | Portworx Essentials | Portworx CSI for FlashArray and FlashBlade | Portworx Enterprise |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------ | ------------------- |
| Number of nodes maximum                                | Defines the maximum number of nodes in a cluster                                                                                                                            | 5                   | 1000                                       | 1000                |
| Number of volumes per cluster maximum                  | Defines max number of volumes on a single node                                                                                                                              | 200                 | 100000 Pure, <br>200 Portworx                   | 100000              |
| Volume capacity \[TB\] maximum                         | Defines max size of a single volume                                                                                                                                         | 1 TB                | 40 TB                                      | 40 TB               |
| Node disk capacity \[TB\] maximum                      | Defines max storage capacity of a single node                                                                                                                               | 1 TB                | 256 TB                                     | 256 TB              |
| Node disk capacity extension                           | Defines is the storage capacity can be extended                                                                                                                             | no                  | yes                                        | yes                 |
| Number of snapshots per volume maximum                 | Number of volume snapshots allowed per single volume                                                                                                                        | 5                   | 5                                          | 64                  |
| Number of attached volumes per node maximum            | Number of attached volumes on a single node                                                                                                                                 | 30                  | 128                                        | 256                 |
| Storage aggregation                                    | Defines if volumes may be aggregated across multiple nodes                                                                                                                  | no                  | no                                         | yes                 |
| Shared volumes                                         | Defines if volumes may be shared w/ other nodes                                                                                                                             | yes                 | yes                                        | yes                 |
| Volume sets                                            | Defines if volumes may be scaled                                                                                                                                            | yes                 | yes                                        | yes                 |
| BYOK data encryption                                   | "Bring your own key" for data encryption                                                                                                                                    | yes                 | yes                                        | yes                 |
| Limit BYOK encryption to cluster-wide secrets          | Limits the use of data-encryption keys to only a cluster-wide secrets                                                                                                       | limited             | limited                                    | unlimited           |
| Resize volumes on demand                               | Defines if volumes can be resized                                                                                                                                           | yes                 | yes                                        | yes                 |
| Snapshot to object store \[CloudSnap\]                 | Defines if cloud-snapshots can be used (for example, volume snapshot to Amazon S3 service)                                                                                          | yes                 | yes                                        | yes                 |
| Number of CloudSnaps daily per volume maximum          | Limits how many cloud-snapshots can customers do (per volume, in a day)                                                                                                     | 1                   | 1                                          | unlimited           |
| Cluster-level migration \[Kube-motion/Data Migration\] | Defines if the Data Migration can be used, see [Data Migration](https://docs.portworx.com/install-with-other/datamigration/) | no                  | no                                         | yes                 |
| Disaster Recovery \[PX-DR\]                            | Enables synchronous and asynchronous DR features (requires 2.1 version or later, needs additional license)                                                                          | no                  | no                                         | (AddOn required)    |
| Autopilot Capacity Management                          | Defines if Autopilot can be used                                                                                                                                            | no                  | no                                         | yes                 |
| OIDC Security                                          | Defines if PX-Security can be used, see [PX-Security on an existing cluster](https://docs.portworx.com/concepts/authorization/install/)            | no                  | no                                         | yes                 |
| Bare-metal hosts                                       | Software may be deployed on commodity hardware                                                                                                                              | yes                 | yes                                        | yes                 |
| Virtual machine hosts                                  | Software may be deployed on VMs (including Amazon EC2, OpenStack Nova)                                                                                                | yes                 | yes                                        | yes                 |
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