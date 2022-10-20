---
title: Install Portworx on bare metal air-gapped Kubernetes cluster 
linkTitle: Air-gapped on bare metal
weight: 100
keywords: Install, on-premises, kubernetes, k8s, air gapped, bare metal
description: How to install Portworx on bare metal air-gapped Kubernetes cluster
noicon: true
disableprevnext: true
aliases:
    - /install-portworx/on-premises/airgapped/airgap-install/
---

Follow the instructions on this page to deploy Portworx and its required packages on a bare metal air-gapped Kubernetes cluster using a private container registry.

## Prerequisites

* You must have a Kubernetes cluster deployed on infrastructure that meets the [minimum requirements](/start-here-installation/) for Portworx.
* You must attach backing storage disks to each worker node.
* You must have dedicated [metadata disks](https://docs.portworx.com/concepts/internal-kvdb/) attached to the worker nodes. 
* The same KVDB device should be present on at least 3 of your nodes, and it should have the same unique device name across all the KVDB nodes.

## Configure your environment 

1. Set an environment variable for the Kubernetes version that you are using:

    ```text 
    KBVER=$(kubectl version --short | awk -F'[v+_-]' '/Server Version: / {print $3}')
    ```
2. Set an environment variable to the latest major version of Portworx:

     ```text
    PXVER=<portworx-version>
     ```

2. On an internet-connected host, download the air-gapped-install bootstrap script for the Kubernetes and Portworx versions that you specified:  

    ```text
    curl -o px-ag-install.sh -L "https://install.portworx.com/$PXVER/air-gapped?kbver=$KBVER"
    ```

3. Pull the container images required for the specified versions:

    ```text
    sh px-ag-install.sh pull
    ```
4. Log in to docker:

    ```text
    docker login <your-private-registry>
    ```

5. Push the container images to a private registry that is accessible to your air-gapped nodes. 
Do not include `http://` in your private registry path:

    ```text 
    sh px-ag-install.sh push <your-registry-path>
    ```

    For example:
    ```
    sh px-ag-install.sh push myregistry.net:5443
    ```

    Example for pushing image to a specific repo:

    ```
    sh px-ag-install.sh push myregistry.net:5443/px-images
    ```

### Create a version manifest configmap for Portworx Operator 

1. Download the Portworx version manifest:

    ```text
    curl -o versions "https://install.portworx.com/$PXVER/version?kbver=$KBVER"
    ```
2. Create a configmap from the downloaded version manifest:

    ```text
    kubectl -n kube-system create configmap px-versions --from-file=versions
    ```
### Install NFS packages for sharedv4 feature

Perform the following to install the NFS package on your host systems so that Portworx can use the sharedv4 feature:

1. Start the repository container as a standalone service in Docker by running the following command:
    ```text
    docker run -p 8080:8080 docker.io/portworx/px-repo:1.0.0
    ```
2. Using a browser within your air-gapped environment, navigate to your host IP address where the above docker image is running (For example, `http://<ip-address>:8080`), and follow the instructions for your Linux distribution provided by the container to configure your host to use the package repository service, and install the NFS packages.

    ![screen capture of the service URL steps](/img/package-repo-screen.png)

## Generate a Portworx spec

1. Navigate to [PX-Central](https://central.portworx.com/).

2. Select **{{< pxEnterprise >}}** from the product catalog.

3. On the **Product Line** page, choose any option depending on which license you intend to use, then click **Continue**.

5. Select the **Use the Portworx Operator** and **Built-in ETCD** options. For **Portworx version**, select the same value from the dropdown that you have set as your Portworx version in the previous section. Click **Next**.

6. Choose your network options and click **Next**.

7. In the **Customize** tab, for the **Are you running on either of these?** option, select **None**. Provide your internal registry path and the details for how to connect to your private registry in **Registry And Image Settings**.. 

    Click the **Finish** button to generate your specs.

    ![Your registry details](/img/private-registery-airgapped.png)

    
## Apply specs

Apply the Operator and StorageCluster specs you generated in the section above by performing the following steps:

1. Deploy the Operator:

    ```text
    kubectl apply -f 'https://install.portworx.com/<PXVER>?comp=pxoperator'
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
    kubectl apply -f 'https://install.portworx.com/<PXVER>?operator=true&mc=false&kbver=&b=true&c=px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```

{{< content "shared/post-installation.md" >}}

