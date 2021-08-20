---
title: Install and configure Portworx on VMware Tanzu
description: Learn to scale a Portworx cluster up or down on VMware Tanzu with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: Install
weight: 3
noicon: true
---

{{<info>}}
**NOTE:** 

* The installation steps below only apply if you're running with Kubernetes on the TKG or TKGS platforms
* [Metro DR](https://docs.portworx.com/portworx-install-with-kubernetes/disaster-recovery/) is currently not supported when both clusters are running TKG
* For the underlying datastore for the PVs and VMs, **do not enable Storage DRS**. VSphere CSI driver and Cloud Native Storage [does not currently support Storage DRS feature in vSphere](https://vsphere-csi-driver.sigs.k8s.io/supported_features_matrix.html)
{{</info>}}

## Prerequisites

* A running Kubernetes cluster TKG or TKGS.
* If you're using TKGS, ensure all prerequisites are met based on [VMware documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-3040E41B-8A54-4D23-8796-A123E7CAE3BA.html#prerequisites-1).
* Enable Workload Management according to [VMware Best Practices](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-3040E41B-8A54-4D23-8796-A123E7CAE3BA.html).
* Ensure the required ports in the `9001:9020`  range are open on all nodes.

## Create the StorageClass

CSI cloud drive configuration requires a StorageClass with the CSI driver set as a provisioner to be installed in the Kubernetes cluster.

1. Get a CSI provisioner name by running the following command on your Tanzu Kubernetes cluster. You will use this name in the next step:
  
    ```text
    kubectl get csidriver
    ```

2. Create a StorageClass. In the `provisioner` field, enter the CSI driver name you got in the step above:

    ```text
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
    annotations:
      storageclass.kubernetes.io/is-default-class: "true"
    name: vsphere-immediate-sc
    provisioner: <csi_driver_name>
    allowVolumeExpansion: true
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    ```

## Install Portworx

As part of the installation, you'll use the spec generator. Perform the following steps to create your Portworx spec and configure your Cloud Drives. This installation method uses the Operator. 

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

 1. Navigate to <a href="https://central.portworx.com" target="tab">PX-Central</a> and log in, or create an account
   
 2. Select **Install and Run** to open the Spec Generator

    ![Screenshot showing install and run](/img/pxcentral-install.png)

 3. Select **New Spec**

    ![Screenshot showing new spec button](/img/pxcentral-spec.png)

 4. Choose either **{{< pxEnterprise >}}** or **{{< pxEssentials >}}**, depending on which license you intend to use:

    ![Screenshot showing Portworx license selector](/img/pxcentral-license.png)

 5. Select VMware Tanzu Cloud Platform and specify disk size and storage class name you will be using for drives

    ![Screenshot showing Tanzu configuration](/img/wmvare-tanzu-configuration.png)


## Open required ports on TKGS

{{<info>}}
**IMPORTANT:** If you aren't installing Portworx on TKGS, skip this section.
{{</info>}}

Portworx requires open ports on the `9001:9020` port range between all nodes. To open these required ports on TKGS, you must SSH into a running node and configure the `iptables` manually.

{{<info>}}
**NOTE:** If you enable autoscaling on your cluster, new nodes will come up with ports closed. 
{{</info>}}

Perform the following steps to open the ports on TKGS nodes:

1. Run the following command from your active terminal:

    ```text
    for pod in $(kubectl get pods -n kube-system -l name=portworx | grep -v NAME | awk '{print $1}');\
    do kubectl exec -t $pod -n kube-system -- nsenter --mount=/host_proc/1/ns/mnt bash -c \
      "iptables -A INPUT -p tcp --match multiport --dports 9001:9020 -j ACCEPT";\
    done
    ```

    Once you've issued this command, the cluster will finish deploying. This usually takes between 5-10 minutes.

2. Watch the pods to ensure Portworx has deployed successfully:

    ```text
    watch kubectl get pods -n kube-system -l name=portworx
    ```

[https://vsphere-csi-driver.sigs.k8s.io/supported_features_matrix.html]: https://vsphere-csi-driver.sigs.k8s.io/supported_features_matrix.html