---
title: Install the monitoring service
weight: 3
keywords: Install, PX-Central, On-prem, license, GUI, k8s, license server
description: Learn how to install the monitoring service.
noicon: true
---

Follow the steps in this section to install the monitoring service.

## Prerequisites

* You must first deploy the PX-Backup chart, and all components should be in running state.

* If you're running Portworx on Openshfift, then you must add a new  service account to the `privileged` SCC. Enter the `oc edit scc privileged` command and add the following line to the `users` section, replacing `<YOUR_NAMESPACE>` with your namespace:

    ```text
    system:serviceaccount:<YOUR_NAMESPACE>:px-monitor
    ```

* Depending on how you wish to access the `px-backup-ui` service, one of the following ports must be open:
  * If you use an Ingress rule or a load balancer endpoint, the HTTP port `80` must be open.
  * If you use the PX-Central endpoint, the port specified in the `spec.type:NodePort` section of the `px-backup-ui` service must be open.

## Prepare air-gapped environments

If your cluster is internet-connected, skip this section. If your cluster is air-gapped, you must pull the monitoring service and related Docker images to either your Docker registry or directly onto your nodes.

1. Run the following command to create an environment variable called `kube_version` and assign your Kubernetes version to it:

    ```
    kube_version=`kubectl version --short | awk -Fv '/Server Version: / {print $3}'`
    ```

2. Pull the following required Docker images onto your air-gapped environment:

    * docker.io/cortexproject/cortex:v1.1.0
    * docker.io/bitnami/cassandra:3.11.6-debian-10-r153
    * docker.io/library/nginx:1.17
    * docker.io/bitnami/consul:1.8.0-debian-10-r24
    * docker.io/pwxbuild/go-dnsmasq:release-1.0.7
    * docker.io/grafana/grafana:7.1.3
    * docker.io/prometheus/prometheus:v2.7.1
    * docker.io/coreos/configmap-reload:v0.0.1
    * docker.io/coreos/prometheus-config-reloader:v0.34.0
    * docker.io/coreos/prometheus-operator:v0.34.0
    * docker.io/portworx/pxcentral-monitor-post-install-setup:1.2.1
    * docker.io/prometheus/memcached-exporter:v0.4.1
    * docker.io/library/memcached:1.5.7-alpine
    * docker.io/library/memcached:1.5.12-alpine
    * docker.io/portworx/px-operator:1.3.1

3. Pull the monitoring service and related Docker images. How you do this depends on your air-gapped cluster configuration:

    * If you have a company-wide docker-registry server, pull the monitoring service and related Docker images from Portworx:

        ```text
        sudo docker pull <required-docker-images>
        sudo docker tag <required-docker-images> <company-registry-hostname>:5000<path-to-required-docker-images>
        sudo docker push <company-registry-hostname>:5000<path-to-required-docker-images>
        ```

    * If you do not have a company-wide docker-registry server, pull the monitoring service and related Docker images from Portworx onto a computer that can access the internet and send it to your air-gapped cluster. The following example sends the Docker image to the air-gapped cluster over ssh:

        ```text
        sudo docker pull <required-docker-images>
        sudo docker save <required-docker-images> | ssh root@<air-gapped-address> docker load
        ```
## Install the monitoring service

1. Generate the install spec through the **License Server and Monitoring** [spec generator](https://central.portworx.com/specGen/px-central-on-prem-wizard), and enter the following information:

    * **PX-Backup UI Endpoint**: The external IP address of the `px-backup-ui` service
    * **OIDC client secret**: Your OIDC client secret. Use either the `kubectl get cm` or the `kubectl get secret` command to retrieve it, replacing `<RELEASE-NAMESPACE>` with your namespace:

        ```text
        kubectl get cm --namespace <RELEASE-NAMESPACE> pxcentral-ui-configmap -o jsonpath={.data.OIDC_CLIENT_SECRET}
        ```

        or

        ```
        kubectl get secret --namespace <RELEASE_NAMESPACE> pxc-backup-secret -o jsonpath={.data.OIDC_CLIENT_SECRET} | base64 --decode
        ```

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

3. Install the monitoring service using either the `--set` flag or the `values.yml` file provided in the **Step 2** section of the **Complete** tab of the spec generator.
