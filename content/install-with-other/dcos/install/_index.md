---
title: Run Portworx with Mesosphere DC/OS
description: Find out how to deploy Portworx and manage the Portworx cluster using DCOS.
keywords: Install, DCOS, DC/OS, Mesosphere, etcd
weight: 1
linkTitle: Install on DCOS
noicon: true
---

{{<info>}}
This document presents the **non-Kubernetes** method of installing Portworx with Mesosphere DC/OS. Please refer to the [Run Portworx with Kubernetes on Mesosphere DC/OS](/install-with-other/dcos/install-dcos-k8s/) page if you want to install Portworx with Kubernetes on Mesosphere DC/OS.
{{</info>}}

This DCOS service will deploy Portworx as well as all the dependencies and additional services to manage the Portworx
cluster. This includes a highly available etcd cluster.

Portworx can be used to provision volumes on DCOS using either the Docker Volume Driver Interface (DVDI) or, directly
through CSI.

{{<info>}}Please ensure that your mesos private agents have unmounted block devices that can be used by Portworx.
{{</info>}}

## (Optional) Deploy an AWS Portworx-ready cluster
Using [this AWS CloudFormation template](/install-with-other/dcos/operate-and-maintain/px-ready-aws-cf), you can easily deploy a
DCOS 1.10 cluster that is "Portworx-ready".

## Pre-install (only required if moving from a Portworx Docker installation)
If you are moving from a Docker install of Portworx to an OCI install, please make sure that the Portworx service is stopped
on all the agents before updating to the OCI install. To do this run the following command on all your private agents:
```text
sudo systemctl stop portworx
```

## Deploy Portworx
The Portworx service is available in the DCOS universe, you can find it by typing the name in the search bar.

![Portworx in DCOS Universe](/img/dcos-px-universe.png)


To modify the defaults, click on the `Review & Run` button next to the package on the DCOS UI.

On the `Edit configuration` page you can change the default configuration for Portworx deployment. Here you can choose to
enable etcd (if you do not have an external etcd service). To have a custom etcd installation please refer to
[this doc](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/etcd).

### Portworx Options
Specify your kvdb (consul or etcd) server if you don't want to use the etcd cluster with this service. If the etcd cluster
is enabled this config value will be ignored.

{{<info>}}
**Note:**<br/>If you are trying to use block devices that already have a filesystem on them, either add the `-f` option
to `portworx options` to force Portworx to use these disks or wipe the filesystem using wipefs command before installing.
{{</info>}}

![Portworx Install options](/img/dcos-px-install-options.png)

{{<info>}}
**Note:**<br/>For a full list of installation options, please look [here](/install-with-other/docker/standalone).
{{</info>}}

### Secrets Options
To use DC/OS secrets for Volume Encryption and storing Cloud Credentials, refer [Portworx with DC/OS Secrets](/key-management/dc-os-secrets).

### Etcd Options
By default a 3 node etcd cluster will be created with 5GB of local persistent storage. The size of the persistent disks can
be changed during install. This can not be updated once the service has been started so please make sure you have enough
storage resources available in your DCOS cluster before starting the install.
![Portworx ETCD Install options](/img/dcos-px-etcd-options.png)

## Install Status

Once you have started the install you can go to the `Services` page to monitor the status of the installation.

If you click on the `portworx` service you should be able to look at the status of the tasks being created. If
you have enabled etcd, there will be 1 task for the framework scheduler and 3 tasks for etcd. Apart from these there will be one task on every node where Portworx runs.

![Portworx Install finished](/img/dcos-px-install-finished.png)

## Scaling Up Portworx Nodes
If you add more agents to your DCOS cluster and you want to install Portworx on those new nodes, you can increase the
`node count` to start install on the new nodes. This will relaunch the service scheduler and install Portworx on the nodes
which didn't have it previously.

![Scale up PX Nodes](/img/dcos-px-scale-up.png)

## Install DCOS Portworx CLI
To install the `dcos portworx` CLI, run the following DCOS CLI command:
```text
dcos package install portworx --cli
```
