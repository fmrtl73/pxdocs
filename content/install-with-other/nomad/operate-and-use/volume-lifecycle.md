---
title: Volume Lifecycle Basics with CSI
linkTitle: Volume lifecycle basics
keywords: Install, Nomad, CSI
description: Instructions on creating and using CSI volumes
weight: 1
series: px-nomad-operate-and-use
---


This section provides instrctions for managing Portworx CSI volumes on Nomad.

## Dynamic volume creation

The following steps will allow you to dynamically provision a Portworx CSI volume: 

1. Create a file named `volume.hcl` with the following content:

    ```text
    id           = "volume-1"
    name         = "database"
    type         = "csi"
    plugin_id    = "portworx"
    capacity_min = "1G"
    capacity_max = "1G"

    capability {
      access_mode     = "single-node-writer"
      attachment_mode = "file-system"
    }
    ```

2. Create a volume using the`volume.hcl` file you just created:

    ```text
    nomad volume create volume.hcl
    ```

3. Verify the volume creation; if successful, the status should be xxxxx:

    ```text
    nomad volume status
    ```

## Registering pre-provisioned volumes

To register a pre-provisioned volume, you must first create a volume on one of the nodes. This example uses the Portworx job to find which alloc to run your commands on. 

1. Get the status of the Portworx job:

    ```text
    nomad job status portworx
    ```

    Grab a running allocation ID from the table listed with the previous command. This example uses `8d76fdfc` from the output:

    ```text
    Allocations
    ID        Node ID   Task Group  Version  Desired  Status   Created    Modified
    8d76fdfc  58c745d0  portworx    0        run      running  10d5h ago  10d5h ago
    a75f6340  e374e687  portworx    0        run      running  10d5h ago  10d5h ago
    ff166433  6e130136  portworx    0        run      running  10d5h ago  10d5h ago
    ```

3. Create a volume by executing the following pxctl command in the `8d76fdfc` allocation:

    ```text
    nomad alloc exec 8d76fdfc /opt/pwx/bin/pxctl volume create volume1
    Volume successfully created: 1055712112955862813
    ```

    Note the volume ID in the command output above. This will be used in a future step. It is `1055712112955862813` in this example.

    {{<info>}}
**NOTE:** If the command above failed with `Access denied token is empty`, you must setup a [pxctl context](/reference/cli/authorization/#create-a-context) on that machine.
    {{</info>}}

1. Create a volume registration file `volume-register.hcl`:

    ```text
    id           = "volume-2"
    name         = "database"
    type         = "csi"
    plugin_id    = "portworx"
    external_id  = "1055712112955862813"

    capability {
      access_mode     = "single-node-writer"
      attachment_mode = "file-system"
    }
    ```

2. Register the pre-provisioned volume with the following command:

    ```text
    nomad volume register volume-register.hcl
    ```


## Shared volume creation

Perform the following steps to dynamically provision a shared Portworx CSI volume: 

1. Create a file `volume-shared.hcl` with the following content:

    ```text
    id           = "volume-2"
    name         = "database"
    type         = "csi"
    plugin_id    = "portworx"
    capacity_min = "1G"
    capacity_max = "1G"

    capability {
      access_mode     = "multi-node-multi-writer"
      attachment_mode = "file-system"
    }
    ```

2. Create a volume using the above file:

    ```text
    nomad volume create volume-shared.hcl
    ```

3. List the volume status to see it:

    ```text
    nomad volume status
    ```

## Run a Nomad job with Portworx volumes

1. Create a nomad job configuration file `job.hcl` with the following contents:

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

4. Create a job with the above configuration:

    ```text
    nomad job run job.hcl
    ```

5. Check the status of your job. It should be running shortly after the image pull has finished:

    ```text
    nomad job status mysql-server
    ```

6. This setup can be cleaned up by stopping the job and and deleting the volume:

    ```text
    nomad job stop mysql-server 
    nomad volume delete volume-1
    ```
