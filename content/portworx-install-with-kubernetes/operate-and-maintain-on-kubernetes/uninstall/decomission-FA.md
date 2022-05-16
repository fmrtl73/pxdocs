---
title: Decommission Portworx nodes with FlashArray Cloud drives
weight: 600
linkTitle: Decommission FlashArray nodes
keywords: 
description: 
---

If you're using a Pure FlashArray as a cloud drive provider, decommissioning or wiping a Portworx node doesn't disconnect or delete the drives from the FlashArray. To fully wipe a Portworx node and remove the data from the FlashArray, you must disconnect the drives on the array and delete them. 

Perform the following steps to decommission Portworx nodes with FlashArray Cloud drives:

1. [Decommission](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/decommission-a-node/) the Portworx node.
2. Access the FlashArray through the UI, CLI, or API and disconnect all of the volumes from the host(s) you just wiped. Refer to the documentation stored on your FlashArray for further instructions. 
3. From the Flasharray, delete the volumes you just disconnected.
4. On the wiped Portworx node(s), enter the `multipath -r` command to remove the now disconnected paths:

    ```text
    multipath -r
    ```

Once you've deleted the volumes from the Flasharray and removed the disconnected paths, you can fully decommission or reuse the node.