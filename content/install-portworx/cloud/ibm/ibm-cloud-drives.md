---
title: Install and configure IBM Cloud Drives
logo: /logos/ibm.png
linkTitle: Install and configure IBM Cloud Drives
weight: 500
keywords: IBM Cloud Kubernetes Service, IBM cloud drive, VPC Gen-2
description: Learn how to install and configure Portworx with IBM Cloud Drives.
noicon: true
---

This document describes how to install and configure Portworx with an IBM Cloud drive.

{{<info>}}**NOTE:**

* Portworx supports a maximum of 3 cloud drives per node at install time. Once Portworx is installed, you can add more disks (up to a maximum of 8 disks per node) to the Portworx storage pool [using the `pxctl service pool expand` command with the IBM Block CSI Driver version 4.4 or newer](/portworx-install-with-kubernetes/storage-operations/create-pvcs/expand-storage-pool/#prerequisites).
* The IBM Cloud drives feature is supported only on VPC Gen2 infrastructure. It is not supported on IBM Classic infrastructure.
{{</info>}}

## Install Portworx

1. Create a IBM VPC Gen-2 IKS or OpenShift cluster.

1. Make sure that for IKS, ports `9001-9020` are open on nodes, and for OpenShift (ROKS), ports 17000-17020 are open on nodes. Portworx requires these ranges to be open on nodes to function normally.

1. Run the following command on your IBM IKS/OpenShift cluster to get your CSI provisioner name:

    ```text
    kubectl get csidriver
    ```

1. Use any of the available StorageClasses on IKS/ROKS cluster; {{<companyName>}} recommends the `ibmc-vpc-block-10iops-tier (default)` StorageClass. Alternatively, you can create a Storage Class with `provisioner` set to the value that was returned by the command in the previous step. For example:

    ```text
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
     annotations:
         storageclass.kubernetes.io/is-default-class: "true"
     name: vpc-immediate-sc
    provisioner: vpc.block.csi.ibm.io
    allowVolumeExpansion: true
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    ```

1. To configure your Portworx spec and Cloud Drives using the Operator, navigate to the [spec generator](https://central.portworx.com) and select **Install and Run**.

1. Select **New Spec**, then choose **Portworx Enterprise**.

1. Select **IBM Cloud Platform** and specify the disk size and storage class name for the drives you will be using.

1. Run the following command to install the Portworx Operator deployment spec:

    ```text
    kubectl apply -f 'https://install.portworx.com/{{% currentVersion %}}?comp=pxoperator'
    ```

1. Apply the generated spec with the following command:

    ```text
    kubectl apply -f px-spec.yaml
    ```

## Configure CSI cloud drives using DaemonSet

1. Enable the CSI cloud drives feature flag by adding the environment variable `ENABLE_CSI_DRIVE=true`.
1. To pass the StorageClass name to Portworx, configure each drive set with a different StorageClass using the `-s` option and the `-kvdb_dev` option:

        ```text
        "-s", "sc=<storage_class_name>,size=<disk_size>"
        ```

        ```text
        "-kvdb_dev", "sc=<storage_class_name>,size=<size>"
        ```

## Uninstall

1. Uninstall Portworx [using the Operator](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/uninstall-operator/) or [using DaemonSet](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/uninstall/).

1. Wipe Portworx drives by deleting VolumeAttachments, PVCs, and PVs. To get VolumeAttachments, run the following command:

    ```text
    kubectl get volumeattachment -n kube-system | grep -v NAME | grep va-pvc
    ```

    Then to delete those VolumeAttachments, run the following command:

    ```text
    kubectl delete volumeattachment <volumeattachment_name>
    ```

    {{<info>}}**NOTE:** If a node is no longer physically present in the cluster, the csi-attacher cannot remove VolumeAttachments. In this case, remove the finalzier first then delete the VolumeAttachment. Use the following command:

```text
kubectl patch volumeattachment  <volumeattachment_name> -n kube-system -p '{"metadata":{"finalizers": []}}' --type=merge && kubectl delete volumeattachment <volumeattachment_name>
```
    {{</info>}}

1. Identify the PVCs to be deleted with the following command:

    ```text
    kubectl get pvc -n kube-system | grep -v NAME | grep px-do-not-delete
    ```

1. Delete the PVCs with the following command:

    ```text
     kubectl delete pvc <pvc_name> -n kube-system
    ```

    {{<info>}}
**NOTE:** Once deleted, PVCs should also remove any associated PVs. You may wish to confirm this manually. If the StorageClass used for provisioning Cloud drives has `retain` set to `reclaimPolicy`, then deleting PVC will not delete actual drive on IBM Cloud Platform. In that case, you can use the `ibmcloud` CLI or IBM Cloud dashboard to delete the volumes.
    {{</info>}}

1. If your StorageClass has `RECLAIM POLICY: Retain`, you must also delete the drives using the IBM Cloud UI.

## Related topics

* [Operate IBM Cloud Drives](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/cloud-drive-operations/ibm/operate-cloud-drives)
