---
title: Uninstall Portworx on VMware Tanzu
description: Learn to scale a Portworx cluster up or down on VMware Tanzu with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: Uninstall
weight: 3
---

To begin uninstalling Portworx from VMware Tanzu, follow the steps in the Portworx uninstall section:

{{<widelink url="/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/uninstall-operator/">}}
Uninstall Portworx using the Operator
{{</widelink>}}

{{<info>}}
**NOTE:** Perform these steps from a machine which has `kubectl` access to your cluster.
{{</info>}}

Once you've completed the uninstall steps, return to this document and wipe the Portworx drives.

## Wipe Portworx drives
When installed on Tanzu, Portworx uses PVCs as the cloud drives. To wipe Portworx completely and detach drives from node, you must delete the following objects:

* VolumeAttachments
* PVC
* PV

### Delete VolumeAttachments

1. Find the VolumeAtachments by running the following `kubectl get` command:

    ```text
    kubectl get volumeattachment -n kube-system | grep -v NAME | grep va-pvc
    ```

2. Once you've identified the VolumeAttachements, Delete them by running the following `kubectl delete` command: 
   
    ```text
    kubectl delete volumeattachment <volumeattachment_name>
    ```

    {{<info>}}
**WARNING:** Wait for Kubernetes to complete the VolumeAttachment deletion and avoid force deleting it. If you force delete the VolumeAttachment, the `csi-attacher` will be unable to detach volumes properly, and physical drives will remain attached to nodes.
    {{</info>}}

{{<info>}}
**NOTE**: if the node is no longer physically present in the cluster, the `csi-attacher` service will not be remove the volume attachments. If this occurs, remove the finalizer first, then delete the VolumeAttachment:

```text
kubectl patch volumeattachment  <volumeattachment_name> -n kube-system -p '{"metadata":{"finalizers": []}}' --type=merge && kubectl delete volumeattachment <volumeattachment_name>
```
{{</info>}}
### Delete PVCs

Once the VolumeAttachment is deleted, you're ready to delete the PVCs:

1. Identify the PVCs:

    ```text
    kubectl get pvc -n kube-system | grep -v NAME | grep px-do-not-delete
    ```

2. Delete the PVCs:

      ```text
      kubectl delete pvc <pvc_name> -n kube-system
      ```

{{<info>}}
**NOTE:** Once deleted, PVCs should also remove any associated PVs. You may wish to confirm this manually. 
{{</info>}}