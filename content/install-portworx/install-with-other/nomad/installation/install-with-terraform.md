---
title: Install with Terraform
linkTitle: Install with Terraform
keywords: Install, Nomad, Terraform
description: Instructions for installing Portworx on Nomad with Terraform.
weight: 200
series: px-install-on-nomad-with-others
noicon: true
aliases:
    - /install-with-other/nomad/installation/install-with-terraform/
---
{{<info>}}
This document presents a **non-Kubernetes** method of installing a Portworx cluster. Please refer to the [Portworx on Kubernetes](/operations/operate-kubernetes/) page if you want to install Portworx on Kubernetes.
{{</info>}}

## Installing

To install with **Terraform**, please use the [Terraform Portworx Module](https://registry.terraform.io/modules/portworx/portworx-instance/)


## Upgrading Portworx

If you have installed Portworx with Terraform, Portworx needs to be upgraded through the CLI on a node-by-node basis. Please see the [upgrade instructions](/operations/operate-other/)

## Scaling

A Portworx cluster is uniquely defined by its `kvdb` and `clusterID` parameters. As long as these are consistent, a cluster can easily scale up in Terraform, by using the same `kvdb` and `clusterID`, and then increasing the instance `count`.
