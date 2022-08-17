---
title: Portworx Essentials license
description: Learn about the Portworx Essential free product offering
keywords: portworx, essentials
weight: 200
aliases: 
    - /concepts/portworx-essentials/
    - /reference/licensing/portworx-essential/
---

{{< pxEssentials >}} is a free Portworx license with limited functionality that allows you to run small production or proof-of-concept workloads. Essentials limits capacity and advanced features, but otherwise functions the same way as the fully-featured {{< pxEnterprise >}} version of Portworx.

The {{< pxEssentials >}} license requires that your clusters be connected to the internet and send usage data to PX-Central. {{< pxEssentials >}} clusters connect with PX-Central once per hour to renew license leases. Lease periods last for 24 hours, ensuring that any temporary interruptions to your connectivity do not impact your cluster.

To get started with {{< pxEssentials >}}, sign up and generate a spec on [PX-Central](https://central.portworx.com/). Licenses for Essentials are generated automatically and activate when your cluster connects to PX-Central.

## {{< pxEssentials >}} available features and limitations

The following tables list what features {{< pxEssentials >}} includes and any limitations it may have:

### {{< pxEssentials >}}

| **Feature** | **Included** |
|----|----|
| Maximum {{< pxEssentials >}} licenses per user  | 1 |
| Maximum clusters per {{< pxEssentials >}} license | 1 |
| Maximum nodes per cluster | 5 |
| Host type included | VMs and bare metal servers |
| Number of attached volumes per node maximum | 30 |
| Host storage capacity limits | 1TB per host |
| Cluster storage capacity limits | 5TB per cluster |
| Online support | [Forum access](https://forums.portworx.com) |
| Telemetry requirement (install/operate) | Users must have telemetry enabled in order to use {{< pxEssentials >}}.|
| Licensing | Users must have a {{< pxEssentials >}} license (available for free at PX-Central)|

{{<info>}}
**Note:** The storage value you set at the time of deployment cannot be expanded until you activate the trial/enterprise license.
{{</info>}}

### PX-Store

| **Feature** | **Included** |
|----|----|
| Container-optimized volumes with elastic scaling (no application downtime) | Included |
| Maximum number of volumes per cluster | 200 |
| Maximum volume size | 1TB |
| Storage performance tiers (high, medium, low) | Included |
| Application-aware I/O tuning | Included |
| Read/Write many across multiple containers | Included |
| Failover across nodes, racks, and availability zones | Included |
| Form storage pools from aggregated volumes across hosts | Included |
| Synchronous Replication <!-- what about async? --> | Included |
| Volume Consistency Groups <!-- not sure what this is --> | Included |

<!-- make columns on the left the same as from the website -->

### Data Management

| **Feature** | **Included** |
|----|----|
|PX-Migrate: Migrate volumes and Kubernetes applications across clusters. | Not included |
| Snapshots of stateful applications | Limited to 5 snapshots per volume|
| Cloudsnap backups to cloud storage | Limited to 1 cloudsnap per day per volume |
| Auto-scaling groups (ASG), including AWS, GCP, Azure. | Included |

### PX-Central

| **Feature** | **Included** |
|----|----|
| Cluster management UI | Single user, single cluster |
| Proactive centralized monitoring | Included |
| Cluster management CLI | Included |
| PX-Central on-premises with license server | Not included |

### Advanced Functionality

| **Feature** | **Included** |
|----|----|
| Encryption with bring-your-own-encryption | Limited to cluster-wide encryption |
| Disaster Recovery for campus and WAN deployments | Not included |
| Capacity Management with Autopilot | Not included |

### Orchestrator

| **Feature** | **Included** |
|----|----|
| Scheduler integration: Kubernetes, Openshift | Included |
| Integration with Managed K8S services: EKS, AKS, GKE, PKS, IKS, RKE | Included |

## The FlashBlade and FlashArray {{< pxEssentials >}} license

If you're using Portworx with FlashBlade or FlashArray, you have access to an enhanced version of the {{< pxEssentials >}} license. Portworx enables this license when it detects the presence of the `pure.json` secret and at least one valid set of credentials.

| **Feature** | **Included** |
|----|----|
| Maximum number of nodes | No limit |
| Number of volumes per cluster maximum | 200 (does not include direct access volumes) |
| Volume capacity [TB] maximum | 40 |
| Node disk capacity extension | yes |
| Number of snapshots per volume maximum | 5 |
| Number of attached volumes per node maximum | 128	|
| Storage aggregation | no (feature upgrade needed)	|
| Shared volumes | yes |
| Volume sets | yes	|
| Limit BYOK encryption to cluster-wide secrets	| yes |
| Resize volumes on demand | yes |
| Snapshot to object store [CloudSnap] | yes |
| Number of CloudSnaps daily per volume maximum | 1	|
| Cluster-level migration [Kube-motion/Data Migration] | no	(feature upgrade needed) |
| Disaster Recovery [PX-DR] | no (feature upgrade needed) |
| Autopilot Capacity Management	| no (feature upgrade needed) |
| Bare-metal hosts | yes |
| Virtual machine hosts | yes |

## Related Topics

Get started with {{< pxEssentials >}} now:

* [Install Portworx](/operations/operate-kubernetes/)
* [Deploy stateful applications on on Kubernetes with Portworx](/operations/operate-kubernetes/application-install-with-kubernetes/)

Find answers to common questions on the {{< pxEssentials >}} FAQ:

* [Portworx FAQ](https://forums.portworx.com/t/portworx-essentials-faq/346)
