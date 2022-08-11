---
title: CSI topology for FlashArray Direct Access volumes and FlashBlade Direct Access filesystems
linkTitle: FlashArray and FlashBlade CSI topology
weight: 400
keywords: cluster topology awareness, kubernetes k8s, geography, locality, rack, zone, region, FlashBlade, FlashArray, csi
description: Learn how to use the CSI topology feature with FlashArray Direct Access volumes and FlashBlade Direct Access filesystems.
noicon: true
series: k8s-op-maintain
---

{{<info>}}
**NOTE:** The CSI topology feature is available only for FlashArray Direct Access volumes and FlashBlade Direct Access filesystems. It cannot be used with FlashArray cloud drives or with other Portworx volumes.
{{</info>}}

The CSI topology feature for FlashArray Direct Access volumes and FlashBlade Direct Access filesystems allows applications to provision storage on a FlashArray Direct Access volume or FlashBlade Direct Access filesystem that is in the same set of Kubernetes nodes where the application pod is located.

## Prerequisites

In order to use the CSI topology feature with a FlashArray Direct Access volume or FlashBlade Direct Access filesystem, you must meet the following prerequisites:

* Install Portworx version 2.11.0 or newer on a local disk or a FlashArray cloud drive. vSphere cloud drives are not supported.
* Install the Portworx Operator version 1.8.1 or newer.
* Install Stork version 2.11 or newer.

## Enable CSI topology

 When you enable CSI topology, you'll specify `Labels` that describe the topology for each FlashArray. The keys must match a set of specific strings, but you can define your own values. The following CSI topology `Labels` keys are available:

* `topology.portworx.io/region`
* `topology.portworx.io/zone`
* `topology.portworx.io/datacenter`
* `topology.portworx.io/provider`
* `topology.portworx.io/row`
* `topology.portworx.io/rack`
* `topology.portworx.io/chassis`
* `topology.portworx.io/hypervisor`

{{<info>}}
**NOTE:** If you are a PSO user, you can use `topology.purestorage.com` labels when you migrate from PSO to Portworx using the pso2px tool.
{{</info>}}

### Enable on a new cluster

{{<info>}}
**NOTE:** To enable the CSI topology feature, you must use the Portworx Operator. You cannot use DaemonSet.
{{</info>}}

To enable the CSI topology feature, perform the following steps:

1. Add the following to your StorageCluster `spec`:

    ```text
        csi:
          enabled: true
          topology:
            enabled: true
    ```

1. Create a `px-pure-secret` containing the information for your FlashArrays. Include `Labels` that specify the topology for each FlashArray. The keys must match a set of specific strings, but you can define your own values. For example:

    ```text
    {
      "FlashArrays": [
        {
          "MgmtEndPoint": "<managementEndpoint>",
          "APIToken": "<apiToken>",
          "Labels": {
            "topology.portworx.io/zone": "zone-0",
            "topology.portworx.io/region": "region-0"
          }
        }
    }
    ```

1. Label your Kubernetes nodes with labels that correspond to the `Labels` from the previous step. For example:

    ```text
    kubectl label node <nodeName> topology.portworx.io/zone=zone-0
    kubectl label node <nodeName> topology.portworx.io/region=region-0
    ```

1. Apply the StorageCluster spec with the following command:

    ```text
    kubectl apply -f <storage-cluster-yaml-file>
    ```

1. Specify the placement strategy by defining the `nodeAffinity` in your Pod or StatefulSet. For example:

    ```text
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.portworx.io/zone
                operator: In
                values:
                - zone-0
              - key: topology.portworx.io/region
                operator: In
                values:
                - region-0
    ```

1. In your StorageClass, choose one of the following strategies so that the PVC uses your topology strategy:

   * Create a StorageClass with [`volumeBindingMode`](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) set to `WaitForFirstConsumer`. For example:

      * FlashArray:

        ```text
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
          name: fio-sc-fada
        provisioner: pxd.portworx.com
        parameters:
          backend: "pure_block"
          max_bandwidth: "10G"
          max_iops: "10G"
          csi.storage.k8s.io/fstype: ext4
        volumeBindingMode: WaitForFirstConsumer
        allowVolumeExpansion: true
        ```

      * FlashBlade:

        ```text
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
          name: fio-sc-fbda
        provisioner: pxd.portworx.com
        parameters:
          backend: "pure_file"
          pure_export_rules: "*(rw)"
        mountOptions:
          - nfsvers=4.1
          - tcp
        volumeBindingMode: WaitForFirstConsumer
        allowVolumeExpansion: true
        ```

   * Create a StorageClass that explicitly defines [`allowedTopologies`](https://kubernetes.io/docs/concepts/storage/storage-classes/#allowed-topologies) in addition to setting [`volumeBindingMode`](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) to `WaitForFirstConsumer`. For example:

      * FlashArray:

        ```text
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
          name: fio-sc-fada
        provisioner: pxd.portworx.com
        parameters:
          backend: "pure_block"
          max_bandwidth: "10G"
          max_iops: "10G"
          csi.storage.k8s.io/fstype: ext4
        volumeBindingMode: WaitForFirstConsumer
        allowedTopologies:
          - matchLabelExpressions:
              - key: topology.portworx.io/rack
                values:
                  - rack-0
                  - rack-1
        ```

      * FlashBlade:

        ```text
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
          name: fio-sc-fbda
        provisioner: pxd.portworx.com
        parameters:
          backend: "pure_file"
          pure_export_rules: "*(rw,no_root_squash)"
        mountOptions:
          - nfsvers=3
          - tcp
        allowVolumeExpansion: true
        allowedTopologies:
          - matchLabelExpressions:
              - key: topology.portworx.io/zone
                values:
                  - zone-1
              - key: topology.portworx.io/region
                values:
                  - c360
        ```

### Enable on an existing cluster

1. Edit the cluster's StorageCluster object to include the following:

    ```text
    csi:
      enabled: true
      topology:
        enabled: true
    ```

1. Delete the existing `px-pure-secret` for your FlashArray or FlashBlade:

    ```text
    kubectl delete secret --namespace kube-system px-pure-secret
    ```

1. Create a new `px-pure-secret` using the following command:

    ```text
    kubectl create secret generic px-pure-secret --namespace kube-system --from-file=<pure.json_file_path>
    ```

    Include `Labels` that specify the topology for each FlashArray or FlashBlade. The keys must match a set of specific strings, but you can define your own values. For example:

    ```text
    {
      "FlashArrays": [
        {
          "MgmtEndPoint": "<managementEndpoint>",
          "APIToken": "<apiToken>",
          "Labels": {
            "topology.portworx.io/zone": "zone-0",
            "topology.portworx.io/region": "region-0"
          }
        }
    }
    ```

1. Label your Kubernetes nodes with labels that correspond to the `Labels` from the previous step. For example:

    ```text
    kubectl label node <nodeName> topology.portworx.io/zone=zone-0
    kubectl label node <nodeName> topology.portworx.io/region=region-0
    ```

1. Restart Portworx on all nodes using the following command:

    ```text
    kubectl label nodes --all px/service=restart
    ```

1. Get all Portworx pods using the following command:

    ```text
    kubectl get pods --namespace kube-system -l name=portworx -o wide
    ```

1. Delete Portworx pods for each node one by one using the following command: 

    ```text
    kubectl delete pods --namespace kube-system <px-pod-name>
    ```

1. Wait for the Portworx pods to come up in the node. You can monitor the pods after deletion using the following command:

    ```text
    kubectl get pods --namespace kube-system -l name=portworx -o wide | grep <node-name>
    ```

1. Delete the Portworx pods for next node. Repeat until Portworx pods are restarted for all nodes.

1. Wait for Portworx pods to be up in all nodes.

1. Validate that topology is enabled in a node by describing `csinode` with the following command:

    ```text
    kubectl describe csinode <node-name>
    ```
    ```output
    Name:               <node-name>
    ...
    Spec:
      Drivers:
        pxd.portworx.com:
          Node ID:        <node-id>
          Topology Keys:  [topology.portworx.io/region topology.portworx.io/zone]
    ```

## Related topics

* [Configure Pure Storage FlashArray as a Direct Access volume](/portworx-install-with-kubernetes/storage-operations/create-pvcs/pure-flasharray)
* [Configure Pure Storage FlashBlade as a Direct Access filesystem](/portworx-install-with-kubernetes/storage-operations/create-pvcs/pure-flashblade)