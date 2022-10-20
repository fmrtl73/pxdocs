---
title: Enable IPv6 with Portworx
keywords: ipv6
description: Instructions for setting up Portworx to accept IPv6 addresses.
linkTitle: IPv6 with Portworx
series: operations-k8s
weight: 1200
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/ipv6/
---
## Supported configurations

IPv6 is supported by Portworx in the following configurations:

| Operating system                    | Kubernetes version | Portworx configuration |
| ----------------------------------- | ------------------ | ---------------------- |
| CentOS 7.9 Kernel: 3.10.0-1160.53.1 | 1.23.0             | <ul><li>Portworx Operator based installation</li><li>Dual network - Management and Data traffic on a separate network</li><li>Air-gapped with all required images pushed to an internal registry</li></ul> |
| CentOS 7.6 Kernel: 5.7.12-1         | 1.20.0             | <ul><li>Portworx Operator based installation</li><li>Single network - Management and Data traffic on a same network</li><li>Air-gapped with all required images pushed to an internal registry</li></ul> |
| Ubuntu 18.04.4 Kernel: v5.3         | 1.23.0             | <ul><li>Portworx Operator based installation</li><li>Single network - Management and Data traffic on a same network</li><li>Air-gapped with all required images pushed to an internal registry</li></ul> |

## Supported features

Portworx supports the following features with IPv6:

* All basic features of Portworx
* Prometheus monitoring
* Autopilot

## Unsupported features

Portworx does not support the following features with IPv6:

* Portworx installation with mixed mode (IPv4 & IPv6) 
* Portworx installation with Cloud Drives
* Portworx installation with proxy
* PX-Central License Server
* PX-Backup
* Asynchronous and Synchronous disaster recovery (DR)
* Telemetry

## Enable IPv6 support

To enable IPv6 support, perform the following steps:

1. Specify your internal registry's Portworx operator image path in your Portworx operator spec. Replace the line `image: portworx/px-operator:<version_number>` in the generated Portworx operator spec with the path to your internal registry operator image. For example:

    ```text
    image: <image_registry_hostname>/<repo_name>/<image_name>:<tag>
    ```

1. If your internal registry is password protected, create a secret to store the credentials, then specify the internal registry access info in the Portworx Storage Cluster spec. For example:

    ```text
    imagePullSecret: <secret_name>
    customImageRegistry: <image_registry_hostname>/<repo_name>
    ```

1. Specify the oci-monitor path in the internal registry. For example:

    ```text
    spec:
      image: <image_registry_hostname>/<repo_name>/<image_name>:<tag>
    ```

1. For dual network deployments, specify network configuration. For example:

    ```text
      network:
        dataInterface: <interface_name_for_data_traffic>
        mgmtInterface: <interface_name_for_mgmt_traffic>
    ```
