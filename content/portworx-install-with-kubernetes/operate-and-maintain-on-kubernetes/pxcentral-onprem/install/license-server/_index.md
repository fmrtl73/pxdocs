---
title: Install the license server component
weight: 2
keywords: Install, PX-Central, On-prem, license, GUI, k8s, license server
description: Learn how to install the license server.
noicon: true
---

Follow the steps in this section to install the license server component.

## Prerequisites

* You must first deploy the PX-Backup chart, and all components should be in running state.
* On a worker node, set the value of the `px/ls` label to `true` by entering the following command:

    ```text
    kubectl label node <NODE_NAME> px/ls=true
    ```
* Enable SSH access on port 7070.

    1. On the node where you've set the value of the `px/ls` label to `true`, add the following line to the `/etc/sysconfig/iptables` file:

        ```text
        -A INPUT -p tcp -m state --state NEW -m tcp --dport 7070 -j ACCEPT in /etc/sysconfig/iptables file
        ```

    2. Restart the iptables service by entering the following command:

        ```text
        systemctl restart iptables.service
        ```

* If you're running PX-Central on Openshfift, then you must add a new  service account to the `privileged` SCC.  Enter the `oc edit scc privileged` command and add the following line to the `users` section, replacing `<YOUR_NAMESPACE>` with your namespace:

    ```text
    system:serviceaccount:<YOUR_NAMESPACE>:pxcentral-license-server
    ```

## Prepare air-gapped environments

If your cluster is internet-connected, skip this section. If your cluster is air-gapped, you must pull the license server and related Docker images to either your Docker registry or directly onto your nodes.

1. Run the following command to create an environment variable called `kube_version` and assign your Kubernetes version to it:

    ```
    kube_version=`kubectl version --short | awk -Fv '/Server Version: / {print $3}'`
    ```

2. Pull the following required Docker images onto your air-gapped environment:

    * docker.io/portworx/px-els:1.0.0
    * docker.io/portworx/pxcentral-onprem-els-ha-setup:1.1.1

3. Pull the license server and related Docker images. How you do this depends on your air-gapped cluster configuration:

    * If you have a company-wide docker-registry server, pull the Portworx license server and related Docker images from Portworx:

        ```text
        sudo docker pull <required-docker-images>
        sudo docker tag <required-docker-images> <company-registry-hostname>:5000<path-to-required-docker-images>
        sudo docker push <company-registry-hostname>:5000<path-to-required-docker-images>
        ```

    * If you do not have a company-wide docker-registry server, pull the license server and related Docker images from Portworx onto a computer that can access the internet and send it to your air-gapped cluster. The following example sends the Docker image to the air-gapped cluster over ssh:

        ```text
        sudo docker pull <required-docker-images>
        sudo docker save <required-docker-images> | ssh root@<air-gapped-address> docker load
        ```
## Install the license server component

1. Generate the install spec through the **License Server and Monitoring** [spec generator](https://central.portworx.com/specGen/px-central-on-prem-wizard).

    If you're using Portworx for the PX-Central installation, select the **Use storage class** checkbox under the **Storage** section of the **Spec Details** tab, and enter the name of the storage class you used to install PX-Central.

    If your cluster is air-gapped, select the **Air Gapped** checkbox, and enter the following information:

      * **Custom Registry**: The hostname of your custom registry
      * **Image Repository**: The path to the required Docker images
      * **Image Pull Secret(s)**: A comma-separated list of your image pull secrets.

2. Using Helm, add the {{< pxEnterprise >}} repo to your cluster and update it:
    <!-- I may instead just push these two steps together and refer users to the spec generator -->

    ```text
    helm repo add portworx http://charts.portworx.io/ && helm repo update
    ```

3. Install the license server component using either the `--set` flag or the `values.yml` file provided in the **Step 2** section of the **Complete** tab of the spec generator.
