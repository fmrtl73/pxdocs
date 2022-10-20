---
title: Install Portworx on bare metal Kubernetes
linkTitle: Bare metal Kubernetes
description: Installation instructions for Portworx Enterprise on Kubernetes running on bare metal infrastructure
keywords: portworx, kurbernetes, containers, storage
weight: 100
---

This article provides instructions for installing Portworx on bare metal Kubernetes clusters using the Portworx Operator.

## Prerequisites

* You must have a Kubernetes cluster deployed on infrastructure that meets the [minimum requirements](https://docs.portworx.com/start-here-installation/) for Portworx.
* You must attach the backing storage disks to each worker node.
* You must have dedicated [metadata disks](https://docs.portworx.com/concepts/internal-kvdb/) attached to the worker nodes. 
* The KVDB device given above needs to be present only on 3 of your nodes and it should have a unique device name across all the KVDB nodes. <!-- does unique here mean the same across all nodes, or a unique name on each node? Also, is this referring to the "metadata disks"? If not, why does it say "given above"? -->


## Install Portworx

### Generate specs

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

1. Navigate to the Portworx [spec generator](https://central.portworx.com/specGen/wizard).


2. Select **{{< pxEnterprise >}}** from the product catalog:

    ![Portworx Operator](/img/install-shared/product-catalog.png)

3. On the **Product Line** page, choose any option depending on which license you intend to use, then click **Continue** to start the spec generator:

    ![Portworx Operator](/img/install-shared/product-line.png)


4. Select **Use the Portworx Operator**, your desired **Portworx version**, and **Built-in ETCD**. Select the **Next** button once you've made your selections:

    <!-- this screenshot shows built-in, do we want to push people to use that?  -->

    ![Basic tab](/img/install-shared/basic.png)


5. Select **On Premises** as your environment, type of **OnPrem storage** <!-- as scan disks? -->, and provide your KVDB device name in the **KVDB Device** field. The KVDB device name should be identical across all the Portworx nodes. <!-- what does this mean? active voice would clear this up. who is performing the action here? the user? Portworx? -->

    <!-- 
    The journaling devices can be ignored on baremetal if the devices used for Data storage is SSD/NVME / Or coming from FC whose back-end is also SSD/NVME storage.

    The minimum kvdb disk size is 64GB and even though we use only 3 node for internal KVDB, It is recommended to have at least 5 nodes with 64GB disk provisioned for seamless KVDB fail-over in case of node failures.
    
    The author missed the Manually specify disk option in the screenshot.
    In this case, the user has to pre-provison the disks to be used with Portworx and KVDB.
    The user-friendly disk names such as /dev/sda should be same across the nodes that will be part of Portworx.

    Automatically Scan disk option is used assuming the user as already assigned the disks that will be used by PX for data storage and KVDB and cannot keep the user friendly names same across all the nodes in the cluster. This also makes user to easily replace/add the disks in future. 
    -->

    ![Storage tab](/img/kubernetes-bare-metal/image2.png)


6. Choose appropriate network options.

    <!-- I need something more than "Choose appropriate network options." -->
    <!-- Data & Mgmt Network can also accept interfaces such as eth0, eth1 OR even bond interfaces such as bond0 and bond1 if user has configured. Details regarding the use of custom Port ranges is missing. -->

    ![Network tab](/img/install-shared/network-default.png)


7. From the **Customize** tab, at the **Are you running on either of these?** dialog, select the **None** radio button. Optionally, change any the following settings: 

    * Registry and image Settings: Allows you to set up your own image registry and details how to connect your private registry.
    * Security Setting: Allows you to setup the cluster with security enabled. This option cannot be enabled if you do not opt for **secure** at the time of installation.
    * Advanced Settings: Gives options to enable CSI, enable monitoring, and give your cluster a unique name.

    ![](/img/kubernetes-bare-metal/image7.png)

    Select the **Finish** button to generate your specs.

Once you've generated your specs, you're ready to apply them and deploy Portworx. You also can save your specs on PX-Central for future reference. 

### Apply specs

Apply the Operator and StorageCluster specs you generated in the section above using the `kubectl apply` command:

1. Deploy the Operator:

    ```text
    kubectl apply -f 'https://install.portworx.com/<version-number>?comp=pxoperator'
    ```
    ```output
    serviceaccount/portworx-operator created
    podsecuritypolicy.policy/px-operator created
    clusterrole.rbac.authorization.k8s.io/portworx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
    deployment.apps/portworx-operator created
    ```

2. Deploy the StorageCluster:

    ```text
    kubectl apply -f 'https://install.portworx.com/<version-number>?operator=true&mc=false&kbver=&b=true&c=px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```

## Monitor Portworx nodes

<!-- these aren't pods, are they? they're storageNode objects. -->

1. Enter the following `kubectl get` command and wait until all Portworx nodes show as `Ready` or `Online` in the output:

    ```text
    kubectl -n kube-system get storagenodes -l name=portworx
    ```
    ```output
    NAME                 ID                                     STATUS   VERSION          AGE
    username-k8s1-node0   7652208b-0bdf-4222-ac83-43cf085e764e   Online   2.11.1-3a5f406   4m52s
    username-k8s1-node1   d43b7ddb-9f2f-4dde-81ff-4597de6fdd32   Online   2.11.1-3a5f406   4m52s
    username-k8s1-node2   0eda7c8b-3f6b-4ce2-b393-e2169ffa111c   Online   2.11.1-3a5f406   4m52s
    ```

2. Enter the following `kubectl describe` command with the `NAME` of one of the Portworx nodes you retrieved above to show the current installation status for individual nodes:

    ```text
    kubectl -n kube-system describe storagenode <portworx-node-name>
    ```
    ```output
    ...
    Events:
        Type     Reason                             Age                     From                  Message
        ----     ------                             ----                    ----                  -------
        Normal   PortworxMonitorImagePullInPrgress  7m48s                   portworx, k8s-node-2  Portworx image portworx/px-enterprise:2.10.1.1 pull and extraction in progress
        Warning  NodeStateChange                    5m26s                   portworx, k8s-node-2  Node is not in quorum. Waiting to connect to peer nodes on port 9002.
        Normal   NodeStartSuccess                   5m7s                    portworx, k8s-node-2  PX is ready on this node
    ```

{{<info>}}
**NOTE:** 

* In your output, the image pulled will differ based on your chosen Portworx license type and version.
* For {{< pxEnterprise >}}, the default license activated on the cluster is a 30 day trial that you can convert to a SaaS-based model or a generic fixed license.
* For {{< pxEssentials >}}, your cluster must have internet connectivity so that it can send usage information every 24 hours to renew the license on the cluster. You can convert an Essentials license to either a fixed license or SaaS-based license.
{{</info>}}

{{< content "shared/post-installation.md" >}}

<!-- {{< content "shared/portworx-install-with-kubernetes-post-install.md" >}} -->

<!-- 
Storage operations
 
Stateful applications on Kubernetes
 
Operate and Maintain 
-->