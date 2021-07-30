---
title: Disk provisioning on Pure Storage FlashArray
description: Use Portworx with Pure Storage FlashArray as a cloud storage provider.
keywords: FlashArray, cloud storage
linkTitle: Pure FlashArray
weight: 1
noicon: true
---

{{<info>}}
**NOTE:** This feature is currently in beta. 
{{</info>}}

You can use Portworx with Pure Storage FlashArray as a cloud storage provider. This allows you to store your data on-premises with FlashArray while benefiting from Portworx cloud drive features, such as:

* Automatically provisioning block volumes
* Expanding a cluster by adding new drives or expanding existing ones
* Support for PX-Backup and Autopilot

Just like with other cloud providers, FlashArray interacts with the underlying drives and distributes volumes between registered arrays, while Portworx runs on top of the volumes that FlashArray presents. 

{{<info>}}
**NOTE:** FlashArray volumes are not the same as Portworx volumes; multiple Portworx volumes can reside on a single FlashArray volume. This makes it theoretically possible for Portworx to place multiple replicas on the same FlashArray volume. 
{{</info>}}
### Architecture

* Portworx runs on each node and forms a storage pool based on configuration information provided in the storageCluster spec.
* When a user creates a PVC, Portworx provisions the volume from the storage pool. 
* The PVCs consume space on the storage pool, and if space begins to run low, Portworx can add or expand drive space from FlashArray.
* If a node goes down for less than 2 minutes, Portworx will reattach FlashArray volumes when it recovers. If a node goes down for more than two minutes, a storageless node in the same zone will take up the volumes and assume the identity of the downed storage node.

![](/img/FaCloudDisk.png)

## Install Portworx and configure FlashArray

Before you install Portworx, ensure that your physical network is configured appropriately and that you meet the prerequisites. You must provide Portworx with your FlashArray configuration details during installation. 

### Configure your physical network

Before you can use Portworx with FlashArray, you must ensure your physical network is appropriately configured. The instructions in this document assume the following configuration details:

* Each FlashArray management IP address can be accessed by each node.
* Your cluster contains an up-and-running FlashArray with an existing dataplane connectivity layout (iSCSI, Fibre Channel).
* If you're using iSCSI, the storage node iSCSI initiators are on the same VLAN as the FlashArray iSCSI target ports.
* If you're using Fibre Channel, the storage node Fibre Channel WWNs have been correctly zoned to the FlashArray Fibre Channel WWN ports. 
* You have an API token for a user on your FlashArray with at least `storage_admin` permissions. Check the documentation on your device for information on generating an API token.

### Prerequisites

* An on-prem kubernetes cluster with FlashArray
* FlashArray must be running a minimum Purity//FA version of at least 4.8
* Latest linux multipath software package installed for your operating system. Refer to the [Linux recommended settings](https://support.purestorage.com/Solutions/Linux/Linux_Reference/Linux_Recommended_Settings) article of the Pure Storage documentation for recommendations.
* If using iSCSI, the latest iSCSI initiator software for your operating system
* If using Fibre Channel, the latest Fibre Channel initiator software for your operating system
* The FlashArray should be time-synced with the same time service as the Kubernetes cluster

### Deploy Portworx

Once you've configured your physical network and ensured that you meet the prerequisites, you're ready to deploy Portworx:

1. Create a JSON file named `pure.json` that contains your FlashArray information:
   
    ```text
    {
      "FlashArrays": [
        {
          "MgmtEndPoint": "<first-fa-management-endpoint>",
          "APIToken": "<first-fa-api-token>"
        }
      ]
    }
    ```

    {{<info>}}
**NOTE:** You can add FlashBlade configuration information to this file if you're configuring both FlashArray and FlashBlade together. Refer to the [JSON file](/reference/pure-json-reference/) reference for more information.
    {{</info>}}

1. Enter the following `kubectl create` command to create a Kubernetes secret called `px-pure-secret`:

    ```text
    kubectl create secret generic px-pure-secret --namespace kube-system --from-file=pure.json
    ```
    ```output
    secret/px-pure-secret created
    ```
    
    {{<info>}}
**NOTE:** You must name the secret `px-pure-secret`.
    {{</info>}}

3. Generate an [install spec](/portworx-install-with-kubernetes/on-premise/other) for your on-prem cluster. Make the following selections when you create your spec using the spec generator:

   * On the **Storage** tab, select the **Cloud** storage environment. <!-- TODO: need UI example from dev. -->
   * Ensure **CSI** is enabled

Once deployed, Portworx detects that the FlashArray secret is present when it starts up and uses the described FlashArray(s) as the storage provider. It then picks a backend for each drive to use, creates volumes, and attaches the volumes using iSCSI or Fibre Channel. 

## Related topics

* [Pure.json file reference](/reference/pure-json-reference/)
* [Configure FlashBlade as a Direct Access filesystem](/portworx-install-with-kubernetes/storage-operations/create-pvcs/pure-flashblade/)
