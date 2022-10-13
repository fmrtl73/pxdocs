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
In addition to managing a Portworx cluster, the Operator also manages the following other components in the Portworx platform:

- Stork
- Autopilot

For simplicity, the Portworx Operator handles the component upgrades without user intervention. When Portworx upgrades, the Operator upgrades the installed components to the recommended version as well.

## Prerequisites

* You must already be running Portworx through the Operator, this method will not work for other Portworx deployments
* You must be running the [latest available version](https://github.com/libopenstorage/operator/releases) of the Operator

## Prepare Anthos environments

*If you're not running Portworx on Anthos, skip this section.* 

If your Portworx cluster is managing the underlying storage devices in an Anthos deployment, add the following annotation to the `StorageCluster` spec to ensure that during an Anthos or a Portworx upgrade, Portworx does not failover the internal KVDB to a storageless node:

```text
portworx.io/misc-args: "-rt_opts wait-before-retry-period-in-secs=360"
```

## Upgrade Portworx

1. Enter the `kubectl get` command to identify your storage cluster name:

      ```text
      kubectl get storagecluster -n kube-system
      ```
      ```
      NAME                                              CLUSTER UUID                           STATUS   VERSION   AGE
      px-cluster-2dc24b8d-dcf5-46d7-b4ea-0477837b54c7   9443277e-ea51-40da-b9b1-7b4739f27cf3   Online   2.10.3    43d
      ```
      {{<info>}}
**NOTE:** If your cluster is not installed in `kube-system`, specify your own namespace with the `-n` flag.
      {{</info>}}

2. Enter the `kubectl edit` command to modify your storage cluster:

      ```text
      kubectl edit storagecluster -n kube-system <storagecluster-name>
      ```

3. Change the `spec.image` value to the version you want to update Portworx to:

      ```text
      apiVersion: core.libopenstorage.org/v1
      kind: StorageCluster
      metadata:
        name: portworx
        namespace: kube-system
      spec:
        image: portworx/oci-monitor:{{<highlight>}}\<your-desired-version\>{{</highlight>}}
      ```
      ```
      storagecluster.core.libopenstorage.org/px-cluster-2dc24b8d-dcf5-46d7-b4ea-0477837b54c7 edited
      ```

      {{<info>}}
**NOTE:** To look up desired version, refer to the [Portworx release notes](/release-notes/portworx/)
      {{</info>}}

4. Verify your upgrade using `kubectl get` command and make note of the version displayed under "VERSION".

      ```text
      kubectl get storagenodes -n kube-system -l name=portworx
      ```
      ```output
      
      "NAME       ID                                     STATUS   VERSION          AGE",
      "node-1-1   2326f09a-73fb-440a-b059-06c81b607226   Online   2.11.3-8a0b7a8   10d",
      "node-1-2   5a21ad81-4db9-4d44-ba30-cf8908228900   Online   2.11.3-8a0b7a8   10d",
      "node-1-3   4b5e3f46-fc14-442e-adc8-c3239f8428d6   Online   2.11.3-8a0b7a8   10d"
      ```
