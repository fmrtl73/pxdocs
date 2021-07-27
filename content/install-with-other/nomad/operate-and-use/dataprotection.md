---
title: Data Protection and Snapshots
linkTitle: Data protection and snapshots
keywords: Install, Nomad, CSI
description: Instructions on creating and using CSI volumes
weight: 3
series: px-nomad-operate-and-use
---

This section provides instructions for taking volume snapshots using the Portworx CSI driver on Nomad.

## CSI Snapshot Lifecycle 

The following steps will allow you to snapshot a Portworx CSI volume: 

1. Create a file `volume.hcl` with the following content:

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
    ```

2. Create a volume using the above file:

    ```
    nomad volume create volume.hcl
    ```

3. Create a snapshot of the above volume:

    ```text
    nomad volume snapshot create volume-1 snap-1
    ```

4. List all snapshots to see the one you've created:

    ```text
    nomad volume snapshot list
    ```

5. Delete the above snapshot:

    ```text
    nomad volume snapshot delete snap-1
    ```


{{<info>}}
**NOTE:** **Snapshots with authorization and authentication enabled**

Due to a few limitions with Nomad, Portworx authorization and authentication will not work with snapshotting. You can track the following issues for information on this support:

* [CSI Volume Snapshot List: support secrets in the request](https://github.com/hashicorp/nomad/issues/10640)
* [CSI Volume Snapshot Create secrets support](https://github.com/hashicorp/nomad/issues/10639)
{{</info>}}