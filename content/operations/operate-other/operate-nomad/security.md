---
title: Secure your volumes with PX Security
linkTitle: Secure your volumes
keywords: Install, Nomad, CSI
description: Instructions on creating and using CSI volumes
weight: 200
series: px-nomad-operate-and-use
aliases:
    - /install-with-other/nomad/operate-and-use/security/
---
This section covers information on utilizing the Portworx CSI driver on Nomad.

## Prerequisites

Be sure to [enable PX security](/install-with-other/nomad/installation/install-as-a-nomad-job/#enable-px-security) when installing Portworx.

## Configure Authorization and Authentication

Perform the following steps to provision and mount volumes with security enabled:

1. Create a file named `volume.hcl` with the following content and replace `<AUTH_TOKEN>` with a JWT token you generated:

    ```text
    id           = "volume-1"
    name         = "database"
    type         = "csi"
    plugin_id    = "portworx"
    capacity_min = "1G"
    capacity_max = "1G"

    capability {
      access_mode     = "single-node-reader-only"
      attachment_mode = "file-system"
    }

    capability {
      access_mode     = "single-node-writer"
      attachment_mode = "file-system"
    }
    secrets {
      auth-token: <AUTH_TOKEN>
    }
    ```


2. Create a volume using the`volume.hcl` file you just created:

    ```text
    nomad volume create volume.hcl
    ```

3. Create a nomad job configuration file named `job.hcl` with the following contents:

    ```text
    job "mysql-server" {
      datacenters = ["dc1"]
      type        = "service"

      group "mysql-server" {
        count = 1

        volume "database" {
          attachment_mode = "file-system"
          access_mode     = "single-node-writer"
          type            = "csi"
          read_only       = false
          source          = "volume-1"
        }

        network {
          port "db" {
            static = 3306
          }
        }

        restart {
          attempts = 10
          interval = "5m"
          delay    = "25s"
          mode     = "delay"
        }

        task "mysql-server" {
          driver = "docker"

          volume_mount {
            volume      = "database"
            destination = "/srv"
            read_only   = false
          }

          env {
            MYSQL_ROOT_PASSWORD = "password"
          }

          config {
            image = "hashicorp/mysql-portworx-demo:latest"
            args  = ["--datadir", "/srv/mysql"]
            ports = ["db"]
          }

          resources {
            cpu    = 500
            memory = 1024
          }

          service {
            name = "mysql-server"
            port = "db"

            check {
              type     = "tcp"
              interval = "10s"
              timeout  = "2s"
            }
          }
        }
      }
    }
    ```

4. Create a job using the `job.hcl` configuration you just created:

    ```text
    nomad job run job.hcl
    ```

5. Check the status of your job:

    ```text
    nomad job status mysql-server
    ```

    It should be running shortly after the image pull has finished:

6. After the installation completes, clean up the setup by stopping the job and and deleting the volume:

    ```text
    nomad job stop mysql-server 
    nomad volume delete volume-1
    ```

{{<info>}}
**NOTE:** **Snapshots with authorization and authentication enabled**

Due to a few limitions with Nomad, Portworx authorization and authentication will not work with snapshotting. You can track the following issues for information on this support:

* [CSI Volume Snapshot List: support secrets in the request](https://github.com/hashicorp/nomad/issues/10640)
* [CSI Volume Snapshot Create secrets support](https://github.com/hashicorp/nomad/issues/10639)
{{</info>}}
