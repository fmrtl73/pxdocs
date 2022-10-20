---
title: Install on Docker Swarm
keywords: Install, Docker, Swarm
description: Learn how to use Portworx to provide storage for your stateful services running on Docker Swarm.
weight: 200
noicon: true
series: px-docker-install
aliases:
    - /install-with-other/docker/swarm/
---
{{<info>}}
This document presents the **Docker** method of installing a Portworx cluster using Docker in swarm mode. Please refer to the [Portworx on Kubernetes](/operations/operate-kubernetes/) page if you want to install Portworx on Kubernetes.
{{</info>}}

This section describes installing Portworx on Docker Swarm.

## Identify storage

Portworx pools the storage devices on your server and creates a global capacity for containers.

{{<info>}}Back up any data on storage devices that will be pooled. Storage devices will be reformatted!
{{</info>}}

To view the storage devices on your server, use the `lsblk` command.

For example:

```text
lsblk
```

```output
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    xvda                      202:0    0     8G  0 disk
    └─xvda1                   202:1    0     8G  0 part /
    xvdb                      202:16   0    64G  0 disk
    xvdc                      202:32   0    64G  0 disk
```

Note that devices without the partition are shown under the **TYPE** column as **part**. This example has two non-root storage devices \(/dev/xvdb, /dev/xvdc\) that are candidates for storage devices.

Identify the storage devices you will be allocating to Portworx. Portworx can run in a heterogeneous environment, so you can mix and match drives of different types. Different servers in the cluster can also have different drive configurations.

## Install

Portworx runs as a container directly via OCI runC. This ensures that there are no cyclical dependencies between Docker and Portworx.

On each swarm node, perform the following steps to install Portworx.

### Step 1: Install the Portworx OCI bundle

{{< content "shared/install-with-other-docker-shared-runc-install-bundle.md" >}}

### Step 2: Configure Portworx under runC

{{<info>}}Specify `-x swarm` in the px-runc install command below to select Docker Swarm as your scheduler.{{</info>}}

{{< content "shared/install-with-other-docker-runc-configure-portworx.md" >}}

### Step 3: Starting Portworx runC

{{< content "shared/install-with-other-docker-shared-runc-enable-portworx.md" >}}


### Adding Nodes

To add nodes to increase capacity and enable high availability, simply repeat these steps on other servers. As long as Portworx is started with the same cluster ID, they will form a cluster.

### Access the pxctl CLI

After Portworx is running, you can create and delete storage volumes through the Docker volume commands or the **pxctl** command line tool.

With **pxctl**, you can also inspect volumes, the volume relationships with containers, and nodes. For more on using **pxctl**, see the [CLI Reference](/reference/cli).

To view the global storage capacity, run:

```text
pxctl status
```

The following sample output of `pxctl status` shows that the global capacity for Docker containers is 128 GB.

```text
pxctl status
```

```output
Status: PX is operational
Node ID: 0a0f1f22-374c-4082-8040-5528686b42be
	IP: 172.31.50.10
 	Local Storage Pool: 2 pools
	POOL	IO_PRIORITY	SIZE	USED	STATUS	ZONE	REGION
	0	LOW		64 GiB	1.1 GiB	Online	b	us-east-1
	1	LOW		128 GiB	1.1 GiB	Online	b	us-east-1
	Local Storage Devices: 2 devices
	Device	Path		Media Type		Size		Last-Scan
	0:1	/dev/xvdf	STORAGE_MEDIUM_SSD	64 GiB		10 Dec 16 20:07 UTC
	1:1	/dev/xvdi	STORAGE_MEDIUM_SSD	128 GiB		10 Dec 16 20:07 UTC
	total			-			192 GiB
Cluster Summary
	Cluster ID: 55f8a8c6-3883-4797-8c34-0cfe783d9890
	IP		ID					Used	Capacity	Status
	172.31.50.10	0a0f1f22-374c-4082-8040-5528686b42be	2.2 GiB	192 GiB		Online (This node)
Global Storage Pool
	Total Used    	:  2.2 GiB
	Total Capacity	:  192 GiB
```

## Post-Install

Once you have Portworx up, take a look below at an example of running stateful Jenkins with Portworx and Swarm!
