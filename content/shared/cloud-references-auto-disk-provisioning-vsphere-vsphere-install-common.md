---
title: Portworx vSphere generic spec generation 
description: Portworx vSphere generic spec generation
keywords: vSphere installation, Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, vSphere ASG, Kubernetes, k8s
hidden: true
---

### Step 1: vCenter user for Portworx

You will need to provide Portworx with a vCenter server user that will need to either have the full Admin role or, for increased security, a custom-created role with the following minimum [vSphere privileges](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-FEAB5DF5-F7A2-412D-BF3D-7420A355AE8F.html):

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

If you create a custom role as above, make sure to select "Propagate to children" when assigning the user to the role.

{{<info>}}**NOTE:** All commands in the subsequent steps need to be run on a machine with `kubectl` access.{{</info>}}

### Step 2: Create a Kubernetes secret with your vCenter user and password

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-secret.md" >}}

### Step 3: Generate the specs

#### vSphere environment details

Export the following environment variables based on your vSphere environment. These variables will be used in a later step when generating the yaml spec.

```text
# Hostname or IP of your vCenter server
export VSPHERE_VCENTER=myvcenter.net

# Prefix of your shared ESXi datastore(s) names. Portworx will use datastores who names match this prefix to create disks.
export VSPHERE_DATASTORE_PREFIX=mydatastore-

# Change this to the port number vSphere services are running on if you have changed the default port 443
export VSPHERE_VCENTER_PORT=443
```

#### Disk templates

A disk template defines the VMDK properties that Portworx will use as a reference for creating the actual disks out of which Portworx will create the virtual volumes for your PVCs.

The template adheres to the following format:

```
type=<vmdk type>,size=<size of the vmdk>
```
- __type__: Supported types are _thin_, _zeroedthick_, _eagerzeroedthick_, and _lazyzeroedthick_
- __size__: This is the size of the VMDK in GiB

The following example will create a 150GB zeroed thick vmdk on each VM:

```text
export VSPHERE_DISK_TEMPLATE=type=zeroedthick,size=150
```
