---
title: Inline volumes with Docker
keywords: nomad, inline volume creation, storage on demand
description: Learn how to use Portworx in order to enable applications to have storage provisioned on demand rathern than pre-provisioned.
weight: 1000
noicon: true
aliases:
    - /install-with-other/nomad/operate-and-use/inline-volume-creation/
---
## Overview

Portworx provides a feature that enables applications to have storage provisioned on demand, rather than requiring storage to be pre-provisioned. This was useful when Nomad did not support CSI (container storage interface) volumes. However, with the support of dynamic [CSI volume creation](/operations/operate-other/operate-nomad/volume-lifecycle/) in Nomad 1.1.0 and Portworx 2.8.0, this method has less robust support in the Nomad ecosystem. 

## Create inline volumes

The feature is also referred to as `inline volume creation`. For more information, refer to the [Inline volume spec](/reference/cli/create-and-manage-volumes) section of the documentation.

Using this feature can be seen in the above example in the `volumes` clause. Note that all relevant Portworx volume metadata can be specified through this mechanism.

## Using inline volumes

You can access _Portworx_ volumes through the Nomad `docker` driver by referencing the `pxd` volume driver:

  ```text
    ...
    task "mysql-server" {
        driver = "docker"
        config {
          image = "mysql/mysql-server:8.0"
          port_map {
            db = 3306
          }
          volumes = [
            "name=mysql,size=10,repl=3/:/var/lib/mysql",
          ]
          volume_driver = "pxd"
      }
      ...
  ```
