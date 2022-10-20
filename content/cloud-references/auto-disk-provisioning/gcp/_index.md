---
title: Disk Provisioning on Google Cloud Platform (GCP)
description: This page describes how to setup a production ready Portworx cluster in a Google Cloud Platform (GCP).
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, GCP, Google Cloud Platform
weight: 200
linkTitle: GCP
noicon: true
---

{{<info>}}If you are running on GKE, visit [Portworx on GKE](/install-portworx/cloud/gcp/gke/).{{</info>}}

The steps below will help you enable dynamic provisioning of Portworx volumes in your GCP cluster.

## Prerequisites

**Key-value store**

Portworx uses a key-value store for its clustering metadata. Please have a clustered key-value database (etcd) installed and ready. For etcd installation instructions refer this [doc](/operations/operate-kubernetes/etcd).

**Firewall**

Ensure ports 9001-9022 are open between the nodes that will run Portworx. Your nodes should also be able to reach the port KVDB is running on (for example etcd usually runs on port 2379).

## Create a GCP cluster

To manage and auto provision GCP disks, Portworx needs access to the GCP Compute Engine API.   There are two ways to do this.

### Using instance privileges

{{< content "shared/gce/instance-role.md" >}}

### Using an account file

{{< content "shared/gce/service-account.md" >}}

This json file needs to be made available on any GCP instance that will run Portworx.  Place this file under a `/etc/pwx/` directory on each GCP instance.  For example, `/etc/pwx/gcp.json`.

## Install

If you used an account file above, you will have to configure the Portworx installation arguments to access this file by way of its environment variables.  In the installation arguments for Portworx, pass in the location of this file via the environment variable `GOOGLE_APPLICATION_CREDENTIALS`. For example, use `-e GOOGLE_APPLICATION_CREDENTIALS=/etc/pwx/gcp.json`.

If you installing on Kuberenetes, you can use a secret to mount `/etc/pwx/gcp.json` into the Portworx Daemonset and then expose `GOOGLE_APPLICATION_CREDENTIALS` as an env in the Daemonset.

Follow [these instructions](/) to install Portworx based on your container orchestration environment.

### Disk template

Portworx takes in a disk spec which gets used to provision GCP persistent disks dynamically.

A GCP disk template defines the Google persistent disk properties that Portworx will use as a reference. There are 2 ways you can provide this template to Portworx.

**1. Using a template specification**

The spec follows the following format:
```text
"type=<GCP disk type>,size=<size of disk>"
```

* __type__: Following two types are supported
    * _pd-standard_
    * _pd-ssd_
* __size__: This is the size of the disk in GB

See [GCP disk](https://cloud.google.com/compute/docs/disks/) for more details on above parameters.

Examples:

* `"type=pd-ssd,size=200"`
* `"type=pd-standard,size=200", "type=pd-ssd,size=100"`


**2. Using existing GCP disks as templates**

You can also reference an existing GCP disk as a template. On every node where Portworx is brought up as a storage node, a new GCP disk(s) identical to the template will be created.

For example, if you created a template GCP disk called _px-disk-template-1_, you can pass this in to Portworx as a parameter as a storage device.

Ensure that these disks are created in the same zone as the GCP node group.

### Limiting storage nodes

{{< content "shared/cloud-references-auto-disk-provisioning-asg-limit-storage-nodes.md" >}}

{{< content "shared/cloud-references-auto-disk-provisioning-asg-examples-gcp.md" >}}
