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
**NOTE:** The CSI topology feature is only available for FlashArray Direct Access volumes and FlashBlade Direct Access filesystems. It cannot be used with FlashArray cloud drives or with other Portworx volumes.
{{</info>}}

The CSI topology feature for FlashArray Direct Access volumes and FlashBlade Direct Access filesystems allows applications to provision storage on a FlashArray Direct Access volume or FlashBlade Direct Access filesystem that is in the same set of Kubernetes nodes where the application pod is located.

## Prerequisites

In order to use the CSI topology feature with a FlashArray Direct Access volume or FlashBlade Direct Access filesystem, you must meet the following prerequisites:

* Install Portworx version 2.11.0 or newer
* Install the Portworx Operator version 1.8.1 or newer
* Install Stork version 2.11 or newer

## Enable CSI topology

 When you enable CSI topology, you'll specify `Labels` that describe the topology for each FlashArray. The keys must match a set of specific strings, but you can define your own values. The following CSI topology `Labels` keys are available:

* `topology.portworx.io/region`
* `topology.portworx.io/zone`
* `topology.portworx.io/datacenter`
* `topology.portworx.io/provider`
* `topology.portworx.io/row`
* `topology.portworx.io/rack`
* `topology.portworx.io/chassis`
* `topology.portworx.io/node`
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

2. Create a `pure-secret` containing the information for your FlashArrays. Include `Labels` that specify the topology for each FlashArray. The keys must match a set of specific strings, but you can define your own values. For example:

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


3. Label your Kubernetes nodes with labels that correspond to the `Labels` from the previous step. For example:

    ```text
    kubectl label node <nodeName> topology.portworx.io/zone=zone-0
    kubectl label node <nodeName> topology.portworx.io/region=region-0
    ```

4. Specify the placement strategy by defining the `nodeAffinity` in your Pod or StatefulSet. For example:

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

5. In your StorageClass, choose one of the following strategies so that the PVC uses your topology strategy:

   * Create a StorageClass with [`volumeBindingMode`](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) set to `WaitForFirstConsumer`. For example:

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

   * Create a StorageClass that defines [`allowedTopologies`](https://kubernetes.io/docs/concepts/storage/storage-classes/#allowed-topologies). For example:

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
        allowedTopologies:
          - matchLabelExpressions:
              - key: topology.portworx.io/rack
                values:
                  - rack-0
                  - rack-1
        ```

### Enable on an existing cluster

1. Edit the cluster's StorageCluster `spec` to include the following:

    ```text
    csi:
      enabled: true
      topology:
        enabled: true
    ```

2. Edit the `pure-secret` for your FlashArray to include topology `Labels` using the following command:

    ```text
    kubectl edit secrets pure-secret
    ```

    Include `Labels` that specify the topology for each FlashArray. The keys must match a set of specific strings, but you can define your own values. For example:

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

3. Get all Portworx pods using the following command:

    ```text
    kubectl get pods -n kube-system -l name=portworx -o wide
    ```

4. Delete Portworx pods for each node one by one using the following command: 

    ```text
    kubectl delete pods -n kube-system <px-pod-name>
    ```

5. Wait for the Portworx pods to come up in the node. You can monitor the pods after deletion using the following command:

    ```text
    kubectl get pods -n kube-system -l name=portworx -o wide | grep <node-name>
    ```

6. Delete the Portworx pods for next node. Repeat until Portworx pods are restarted for all nodes.

7. Wait for Portworx pods to be up in all nodes.

8. Validate that topology is enabled in a node by describing `csinode` with the following command:

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