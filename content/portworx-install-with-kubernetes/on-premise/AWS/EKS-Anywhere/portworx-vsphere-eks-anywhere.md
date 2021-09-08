---
title: Install Portworx on vSphere with Amazon EKS Anywhere
linkTitle: Amazon EKS Anywhere
description: This section describes lists the steps to deploy Portworx on vSphere having Amazon EKS Anywhere.
keywords: portworx, VMware, vSphere, EKS Anywhere, Amazon EKS Anywhere
---


Portworx can be installed on a Kubernetes cluster running on vSphere and managed by EKS Anywhere.

### Prerequisites

Before you install Portworx on vSphere, ensure that you meet the following prerequisiites:

|**Specification**| **Supported Value**|
|------|------|
|Deployment Host<br/>**Note**: The same vSphere host where you deploy EKS-Anywhere.| VM OS: Linux <br/>vCPU: 4 <br/> Memory: 16 GB <br/> Disk storage: 200 GB|
|Control-Plane VMs| Minimum: 1 <br/> Recommended: 3 <br/>vCPUs: 2 <br/> RAM: 8 GB <br/> OS Volume: 25 GB|
|Worker Node VMs| Minimum: 3 (for storage cluster) <br/> vCPUs: 8 <br/>RAM: 16 GB <br/> OS Volume: 25 GB|

### Step 1: vCenter user for Portworx

Provide Portworx with a vCenter server user that has either the full Admin role or, for increased security, a custom-created role with the following minimum [vSphere privileges](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-FEAB5DF5-F7A2-412D-BF3D-7420A355AE8F.html):

* Datastore
    * Allocate space
    * Browse datastore
    * Low level file operations
    * Remove file
* Host
    * Local operations
    * Reconfigure virtual machine
* Virtual machine
    * Change Configuration
    * Add existing disk
    * Add new disk
    * Add or remove device
    * Advanced configuration
    * Change Settings
    * Extend virtual disk
    * Modify device settings
    * Remove disk

If you created a custom role with the permissions above,  select "Propagate to children" when assigning the user to the role.

{{<info>}}
**NOTE:** All commands in the subsequent steps need to be run on a machine with kubectl access.
{{</info>}}

### Step 2: Create a Kubernetes secret with your vCenter user and password

{{% content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-secret.md" %}}

### Step 3: Generate specifications

Generate the spec file using the Portworx [Spec Generator](https://central.portworx.com/specGen/wizard) with the following configurations:

1. Under the **Basic** tab, ensure that the following config parameters are set.
    - Select the **Use the Portworx operator** checkbox.
    - Select the **2.8** version of Portworx in **Portworx Version** drop-down. 
    - Under **ETCD**, select **Built-in**.

2. Under the **Storage** tab, ensure that the following config parameters are set.
    - Under **Select your environment**, choose **Cloud**.
    - Under **Select Cloud platform**, select **vSphere**.
    - Under **Configure storage devices**, choose **Create Using a Spec** in **Select type of disk**.
    - Under **vCenter datastore prefix**, type "px-".

3. Under the **Customize** tab, ensure that the following config parameters are set.
    - In **Are you running either of these?**, choose **None**.
    - In **Advanced settings**, select the following checkboxes. 
        - **Enable Stork**
        - **Enable CSI**
        - **Enable Monitoring**
        - **Enable Telemetry**


{{% content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" %}}

