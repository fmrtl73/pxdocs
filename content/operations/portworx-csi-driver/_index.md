---
title: Portworx CSI Driver
description: Find out more about the Portworx CSI Driver and discover documentation for the different container schedulers we support
keywords: portworx, kurbernetes, containers, storage, csi
weight: 400
hidesections: true
noicon: true
disableprevnext: true
scrollspy-container: false
series: operations
aliases:
    - /portworx-csi-driver/
---
The Portworx CSI Driver has been verified on both [Kubernetes](/operations/operate-kubernetes/storage-operations/csi/) and [Nomad](/install-with-other/nomad/). The CSI Driver supports all existing Portworx features as well as most CSI features. See the table below for a detailed picture of what features we support for each scheduler.

For scheduler-specific information, refer to the following pages:

* [CSI Driver on Kubernetes](/operations/operate-kubernetes/storage-operations/csi/)
* [CSI Driver on Nomad](/install-with-other/nomad//)

## Core Features support
| **Feature**  | **Kubernetes** | **Nomad** |
|----------|------------|-------|
| Volume Lifecycle               | Yes                                    | Yes                                                                                                                                                         |
| Sharedv4 Volumes               | Yes                                    | Yes                                                                                                                                                         |
| Volume Cloning                 | Yes                                    | Yes                                                                                                                                                         |
| CSI Snapshotting               | Yes                                    | Yes                                                                                                                                                         |
| CSI Spec 1.4                   | Yes                                    | Yes                                                                                                                                                         |
| PX Security - Volume Lifecycle | Yes                                    | Yes                                                                                                                                                         |
| PX Security - Snapshotting     | Yes                                    | Not currently supported in Nomad: [#10639](https://github.com/hashicorp/nomad/issues/10639) and [#10640](https://github.com/hashicorp/nomad/issues/10640) |
| PX Topology                    | Yes                                    | No                                                                                                                                       |
| CSI Raw Block                  | Yes                                    | No                                                                                                                                           |
| Volume Resizing                | Yes                                    | Not currently supported in Nomad: [#10324](https://github.com/hashicorp/nomad/issues/10324)                                                                |
| Encryption - Vault             | Yes                                    | Not currently supported in Nomad: [#7978](https://github.com/hashicorp/nomad/issues/7978)                                                                  |
| CSI Topology                   | No, PX topology is recommended instead | No                                                                                                                                                          |

## Kubernetes specific features

| **Feature**                         | **Supported**                                                              |
|-------------------------------------|----------------------------------------------------------------------------|
| Volume Placement Strategies             | Yes                                                                    |
| Auth Token Secrets                      | Yes                                                                    |
| Autopilot                               | Yes                                                                    |
| Generic Ephemeral Volumes               | Yes                                                                    |
| Encryption with K8s secrets             | Yes                                                                    |
| CSI Migration (Portworx in-tree -> CSI) | No - coming soon                                                       |

## Nomad specific features

| **Feature**                         | **Supported**                                                              |
|-------------------------------------|----------------------------------------------------------------------------|
| Pre-provisioned volume registration | Yes                                                                        |

## PX-Backup & Stork

| **Feature**                         | **Kubernetes**                                                              | **Nomad** |
|-------------------------------------|--------------------------------------------------|--------------------------------------|
| Stork Snapshotting   | Yes                                                     | No |
| Stork Migration      | Yes                                                     | No |
| Stork Backup         | Yes                                                     | No |
| PX-Backup support    | Yes                                                     | No |
| DR/Disaster Recovery | Yes                                                     | No |

## Installation

| **Feature**                         | **Kubernetes**                                                              | **Nomad** |
|-------------------------------------|--------------------------------------------------|--------------------------------------|
| Operator                     | Yes                                                     | No     |
| Portworx Daemonset Installer | Yes                                                     | No           |
