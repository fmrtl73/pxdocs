---
title: Portworx on Nomad
linkTitle: Nomad
keywords: Install, Nomad
description: Instructions on installing Portworx on Nomad
weight: 400
series: px-other
---

{{<info>}}
This document presents the **Nomad** method of installing a Portworx cluster. Please refer to the [Portworx on Kubernetes](/operations/operate-kubernetes/) page if you want to install Portworx on Kubernetes.
{{</info>}}

This section covers information on Portworx on Nomad.

Nomad is a scheduler and job orchestrator from HashiCorp for managing a cluster of machines and running applications on them. Nomad abstracts away machines and the location of applications and instead enables users to declare what they want to run and Nomad handles where they should run and how to run them. Portworx can run within Nomad and provide persistent volumes to other applications running on Nomad. This section describes how to deploy and consume Portworx within a Nomad cluster.

## Portworx as a Nomad job

These sections explain how to install, upgrade, and uninstall Portworx using a Nomad job:

{{<homelist series="px-as-a-nomad-job">}}

## Operate and utilize Portworx on Nomad

{{<homelist series="px-nomad-operate-and-use">}}

## Open Source

[Nomad](https://github.com/hashicorp/nomad) is an open source project developed by HashiCorp and a community of developers. CSI support for Nomad is still in active development with [many open issues](https://github.com/hashicorp/nomad/issues?q=is%3Aissue+is%3Aopen+csi). Portworx participates in and encourages open source contributions to [Nomad](https://github.com/hashicorp/nomad) as well as the [CSI spec](https://github.com/container-storage-interface/spec).