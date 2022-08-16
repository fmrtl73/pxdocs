---
title: Install with Ansible
linkTitle: Install with Ansible
keywords: Install, Nomad, Ansible
description: Instructions for installing Portworx on Nomad with Ansible.
weight: 100
series: px-install-on-nomad-with-others
noicon: true
aliases:
    - /install-with-other/nomad/installation/install-with-ansible/
---
{{<info>}}
This document presents a **non-Kubernetes** method of installing a Portworx cluster. Please refer to the [Portworx on Kubernetes](/operations/operate-kubernetes/) page if you want to install Portworx on Kubernetes.
{{</info>}}


## Installing

To install with **Ansible**, please use the [Ansible Galaxy Role](https://galaxy.ansible.com/portworx/portworx-defaults/)

## Upgrading

If you have installed Portworx with Ansible, Portworx needs to be upgraded through the CLI on a node-by-node basis. Please see the [upgrade instructions](/operations/operate-other/)

## Scaling

For Ansible, as long as the same `kvdb` and `clusterID` are used, any new nodes can automatically join an existing cluster. Also, be sure to exclude existing nodes from the inventory before running the playbook on the new nodes.
