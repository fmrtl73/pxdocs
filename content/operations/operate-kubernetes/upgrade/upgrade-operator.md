---
title: Upgrade Portworx on Kubernetes using the Operator
linkTitle: Upgrade using the Operator
keywords: portworx, container, kubernetes, storage, k8s, upgrade
description: Find out how to upgrade Portworx using the Operator.
weight: 200
aliases:
  - /portworx-install-with-kubernetes/on-premise/openshift/operator/upgrade/
  - /portworx-install-with-kubernetes/openshift/operator/upgrade
  - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/upgrade/upgrade-operator/
---

{{<info>}}
**WARNING:** If you're upgrading OpenShift to 4.3, you must change Portworx before you can do so. See the [Preparing Portworx to upgrade to OpenShift 4.3](/install-portworx/openshift/operator/openshift-upgrade) page for details.
{{</info>}}

If you're using the Portworx Operator, you can upgrade or change your Portworx version at any time by modifying the `StorageCluster` spec.

## Prerequisites

* You must already be running Portworx through the Operator, this method will not work for other Portworx deployments
* You must be running the [latest available version](https://github.com/libopenstorage/operator/releases) of the Operator

## Upgrade Portworx

{{<info>}}
  **NOTE:** When Portworx is managing the underlying storage devices in an Anthos deployment, add the following annotation to the `StorageCluster` spec:

  ```text
  portworx.io/misc-args: "-rt_opts wait-before-retry-period-in-secs=360"
  ```

  This annotation ensures that during an Anthos or a Portworx upgrade, Portworx does not failover internal KVDB to a storageless node. 
{{</info>}}

1. Enter the `kubectl edit` command to modify your storage cluster:

      ```text
      kubectl edit -n kube-system <storagecluster_name>
      ```

2. Change the `spec.image` value to the version you want to update Portworx to:

      ```text
      apiVersion: core.libopenstorage.org/v1
      kind: StorageCluster
      metadata:
        name: portworx
        namespace: kube-system
      spec:
        image: portworx/oci-monitor:<your_desired_version>
      ```

## Upgrade Portworx components

In addition to managing a Portworx cluster, the Operator also manages the following other components in the Portworx platform:

- Stork
- Autopilot

For simplicity, the Portworx Operator handles the component upgrades without user intervention. When Portworx upgrades, the Operator upgrades the installed components to the recommended version as well.

The Portworx Operator refers to the [release manifest](https://install.portworx.com/version) to determine which recommended component version to install for a given Portworx version. This release manifest is regularly updated for
every Portworx release.

{{<info>}}
**NOTE:** {{<companyName>}} does __not__ recommend you update individual component versions unless absolutely necessary.
{{</info>}}

### Force upgrade Stork

To override the operator selected Stork image, edit the `StorageCluster` object and
modify the `spec.stork.image` field, entering your desired Stork version

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: portworx
  namespace: kube-system
spec:
  stork:
    enabled: true
    image: openstorage/stork:<your_desired_stork_version>
```

### Force upgrade Autopilot

To override the operator selected Autopilot image, edit the `StorageCluster` object and
modify the `spec.autopilot.image` field, entering your desired Autopilot version

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: portworx
  namespace: kube-system
spec:
  autopilot:
    enabled: true
    image: portworx/autopilot:<your_desired_autopilot_version>
```
