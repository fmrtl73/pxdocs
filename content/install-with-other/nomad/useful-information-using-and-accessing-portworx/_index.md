---
title: Using and accessing Portworx on Nomad
linkTitle: Using and accessing
keywords: nomad, access volumes, pxd
description: Learn how to use and access Portworx on Nomad.
weight: 1
series: px-nomad-useful-information
noicon: true
hidden: true
---

_Portworx_ volumes can be easily accessed through the Nomad `docker` driver by referencing the `pxd` volume driver.

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
