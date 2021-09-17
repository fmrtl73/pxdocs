---
title: Troubleshooting
description: Learn how to address common setup errors and collect logs
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, Tanzu, vSphere ASG, Kubernetes, k8s, troubleshooting
linkTitle: Troubleshooting
weight: 3
---


## Troubleshooting
### Common errors
#### The CSI driver failed to bind a PVC to a PV

You can identify this error in the following ways:

* When checking the status of a PVC using `kubectl get pvc -n kube-system`, the PVC displays as in the `pending` state.
* Logs contain the following message: 

    ```
    error level=error msg="Could not init boot manager" error="failed to generate a new node identity: create CSI volume (CreateVolume) error: PVC: kube-system/px-do-not-delete-... failed to get bound to a PV"
    ```

To correct this error, check if the CSI driver exists in your cluster by running the following command:
  
```text
kubectl get csidriver
```
```output
NAME                     ATTACHREQUIRED   PODINFOONMOUNT   MODES        AGE
csi.vsphere.vmware.com   true             false            Persistent   6d22h
```

* If the driver is not present in the system, contact your cluster administrator to install the vSphere CSI driver.
* If the CSI driver exists, check the StorageClass name you’re using for installation. Ensure that StorageClass has the CSI driver name set as a provisioner.

### Collect logs

* Collect portworx pods logs:

    ```
    kubectl logs --since=0 -n kube-system -l name=portworx
    ```

* Find the `vsphere-CSI-controller-***` pod in your system:

    ```text
    kubectl get pods --all-namespaces | grep vsphere-csi-controller
    ```

    Once found, collect CSI driver logs:

    ```text
    kubectl logs --since=0 -n kube-system <vsphere-csi-controller-pod> csi-attacher > csi_attacher.log

    kubectl logs --since=0 -n kube-system <vsphere-csi-controller-pod> vsphere-csi-controller > vsphere-csi-controller.log

    kubectl logs --since=0 -n kube-system <vsphere-csi-controller-pod> csi-provisioner > csi-provisioner.log

    ```

* Collect PVC, PV and VolumeAttachments lists:

    ```text
    kubectl get pvc -n kube-system > pvc.log

    kubectl get pv > pv.log

    kubectl get volumeattachment > volumeattachment.log

    ```
