---
title: "Prerequisites"
weight: 100
disableprevnext: true
equalizeTableWidths: "50%"
description: Start your installation.
keywords: Portworx, containers, storage
aliases:
    - /start-here-installation/
---

## Installation Prerequisites

The minimum supported size for a Portworx cluster is three nodes. Each node must meet the following hardware, software, and network requirements:

|**Hardware** ||
|-------------------------|------------|
|     CPU | 4 cores minimum, 8 cores recommended |
|     RAM | 4GB minimum, 8GB recommended |
| Disk <ul><li>`/var`</li><li>`/opt`</li></ul> | <ul><li>2GB free</li><li>3GB free</li></ul> |
|Backing drive | 8GB (minimum required)<br/>128 GB (minimum recommended)|
|Storage drives | Storage drives must be unmounted block storage: raw disks, drive partitions, LVM, or cloud block storage. |
|Network connectivity | Bandwidth: <ul><li>10 Gbps recommended</li><li>1 Gbps minimum</li></ul><br/>Latency requirements for synchronous replication: less than 10ms between nodes in the cluster|

|**Network** ||
|--- | ---|
|Open needed ports | Portworx requires different open ports depending on how it's installed:<ul><li>Spec-based installations require all Portworx nodes to have open TCP ports at 9001-9022 and an open UDP port at 9002.</li><li>Portworx on OpenShift 4+ requires open TCP ports at 17001-17020 and an open UDP port at 17002.</li></ul>Portworx also requires an open KVDB port. For example, if you're using `etcd` externally, open port 2379.<br/><br/>If you intend to use Portworx with sharedv4 volumes, you may need to [open your NFS ports](/portworx-install-with-kubernetes/storage-operations/create-pvcs/open-nfs-ports).|

|**Software** ||
|--- | ---|
|Linux kernel | Version 3.10 or greater.|
|Docker | Version 1.13.1 or greater.|
|Key-value store | Portworx needs a key-value store to perform its operations. As such, install a clustered key-value database \(`etcd` or `consul`\) with a three node cluster.<br><br>With Portworx 2.0 or greater, you can use Internal KVDB during installation. In this mode, Portworx will create and manage an internal key-value store (kvdb) cluster.<br><br>If you plan of using your own etcd, refer to [Etcd for Portworx](/reference/etcd) for details on recommendations for installing and tuning etcd.|
|Disable swap|Please disable swap on all nodes that will run the Portworx software.  Ensure that the swap device is not automatically mounted on server reboot.|

## Supported Kubernetes versions

Before you install Portworx on Kubernetes, ensure that you're using a supported Kubernetes version:

| **Type** | **Supported Kubernetes Version** |
|---|---|
| On-prem Kubernetes | <ul><li>1.19</li><li>1.20</li><li>1.21</li><li>1.22</li><li>1.23</li></ul> |
| Managed Kubernetes | <ul><li>**KOPS:** 1.21</li><li>**GKE:** 1.22</li><li>**AKS:** 1.21 & 1.22</li><li>**EKS:** 1.21</li><li>**IKS:** 1.19</li><li>**PKS:** 1.17</li></ul> |
| Distribution Kubernetes | <ul><li>**Openshift 4.6:** 1.19 (Openshift version verified up to 4.6.48) </li><li>**Openshift 4.7:** 1.20 (Openshift version verified up to 4.7.37)</li><li>**Openshift 4.8:** 1.21 (Openshift version verified up to 4.8.20) </li><li>**Openshift 4.9:** 1.22 (Openshift version verified up to 4.9.7)</li><li>**Openshift 4.10:** 1.23 (Openshift version verified up to 4.10.9)</li></ul> |
| vSphere with Tanzu (TKGs) | <ul><li>v1.19.14+vmware.1-tkg.1.8753786</li><li>v1.20.9+vmware.1-tkg.1.a4cee5b</li></ul> |
| Tanzu Kubernetes Grid Integrated (TKGI) | <ul><li>PKS Version: 1.9.6-build.7; k8s Version: 1.18.18</li><li>PKS Version: 1.10.7-build.6; k8s Version: 1.19.16</li><li>PKS Version: 1.12.1-build.10; k8s Version: 1.21.5</li><li>PKS Version: 1.13.2-build.16; k8s Version: 1.22.6</li></ul> | 

{{<info>}}
**K3s users:** You must use CSI integration to generate / use PVCs.
{{</info>}}

## Air-gapped prerequisites

If you intend to use the `sharedv4` feature, your host systems must be running the NFS service. For more information on options for installing the NFS service, refer to the installation article for air-gapped clusters. 

## Installation

Whether you're using {{< pxEnterprise >}} or Essentials, you can install Portworx on the cloud or on-premises. Proceed to one of the following sections for Kubernetes and OpenShift install instructions.

{{<homelist series="k8s-install">}}

For all other environments, continue to the following section:

{{< widelink url="/install-with-other" >}}Portworx on other orchestrators{{</widelink>}}

## Post-installation

If you have an existing Portworx cluster, continue to below sections for using and managing Portworx.

{{<homelist series2="k8s-postinstall">}}