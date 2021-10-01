---
title: Tune performance
keywords: performance tuning, optimize
description: Improve the performance of your Portworx cluster on Kubernetes
---

In its default configuration, Portworx attempts to provide good performance across a wide range of situations. However, you can improve your storage performance on Kubernetes by configuring a number of settings and leveraging features Portworx offers. To get the most out of Portworx, follow the guidance provided in this article. 

## Prerequisites

* Portworx 2.7.1 or greater

## Configure the network data interface

You can provide Portworx with a specific network interface for data when you [generate the spec](/portworx-install-with-kubernetes/#installation) as part of your installation. {{<companyName>}} recommends a network interface with a bandwidth of at least 10Gb/s and network latency below 5 milliseconds. If multiple NICs are present on the host, present a bonded interface to Portworx.

{{<info>}}
**NOTE:** If you've already installed Portworx, you can update the `network.dataInterface` value of the install spec and reapply it. 
{{</info>}}

## Enable hyperconvergence 

Use Stork to ensure your Pod is running on the [same node in which the data resides](/portworx-install-with-kubernetes/storage-operations/hyperconvergence/).

## Configure your cluster topology

When configured to be aware of your cluster topology, Portworx places replicas for high availability. [Configure your cluster topology](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/cluster-topology/).

## Define a VolumePlacementStrategy

StatefulSets, distributed NoSQL databases, such as Cassandra, require PVCs to be distributed across the cluster. Use Affinity/Anti Affinity rules along with topology labels to define relationships between PVCs.

<!-- StatefulSets FOR distributed NoSQL databases ?-->

[Define a VolumePlacementStrategy](/portworx-install-with-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/) using affinity and anti-affinity labels to distribute volumes.

## Adjust storage classes

To improve performance, adjust storage class parameters in the following ways:

* Prioritize volume traffic by setting the `priority_io:` field to `high`
* Choose the replication factor best suited to your high availability needs

    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1beta1
    metadata:
     name: px-storage-class
    provisioner: kubernetes.io/portworx-volume
    allowVolumeExpansion: true
    parameters:
     repl: "2"
     priority_io: "high"
     nodiscard: "true"
    ```

## Modify Portworx resource consumption

By default, Portworx consumes as little CPU and memory resources as possible. You can potentially improve performance by allocating more resources, allowing Portworx to use more CPU threads and memory. Do this by modifying the spec used to install Portworx based on your cluster architecture.

### Disaggregated architecture

In disaggregated deployments with dedicated storage nodes, enable higher resource consumption by specifying the `rt_ops_conf_high` runtime option: 
  
* If you installed Portworx using the DaemonSet, enter the runtime option as an argument:
  
      ```text
      "-rt_opts", "rt_opts_conf_high=1"
      ```

      The full arguments section should look like this:

      ```
      args:
                  ["-c", "portworx-k8s-poc-12aa34aa-56e7-8901-b234-b5678901aa23", "-secret_type", "k8s", "-b", 
                  "-x", "kubernetes", "-rt_opts", "rt_opts_conf_high=1, <snip>
      ```   

* If you installed Portworx using the Operator:
  
      ```text
      apiVersion: core.libopenstorage.org/v1
      kind: StorageCluster
      metadata:
          name: px-cluster
          namespace: kube-system
      spec:
          image: portworx/oci-monitor:2.7.0
          ...
          runtimeOptions:
              rt_opts_conf_high: "1"
      ```

### Hyperconverged architecture

In a non-disaggregated/hyperconverged architecture, where applications are running on the same host as storage, set threads based on the number of cores that can be allocated to Portworx. For example, if your host has 16 cores: 

  * `num_threads=16` sets the total number of threads.
  * `num_io_threads=12` sets the number of threads that can do IO out of the total `num_threads`. In general, IO threads should be 75% of total threads.
  * `num_cpu_threads=16` sets the number threads that can do CPU work out of the total `num_threads`.

Configure these values as runtime options:
  
  
* If you installed Portworx using the DaemonSet, enter the values as arguments:

    ```text
    "-rt_opts", "rt_opts_conf_high=1,num_threads=16,num_io_threads=12,num_cpu_threads=16"
    ```

    The full arguments section should appear as follows:

    ```text
    args:
                ["-c", "portworx-k8s-poc-12aa34aa-56e7-8901-b234-b5678901aa23", "-secret_type", "k8s", "-b", 
                "-x", "kubernetes", "-rt_opts", "rt_opts_conf_high=1,num_threads=24,num_io_threads=16,num_cpu_threads=24" , <snip>
    ```   

* If you installed Portworx using the Operator, enter the values into the `runtimeOptions` spec field:

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
        name: px-cluster
    namespace: kube-system
    spec:
        image: portworx/oci-monitor:2.7.0
        ...
        runtimeOptions:
            rt_opts_conf_high: "1"
            num_threads: "16"
            num_io_threads: "12"
            num_cpu_threads: "16"
    ```


To see more StorageCluster examples, visit the [StorageCluster](https://docs.portworx.com/reference/crd/storage-cluster/#storagecluster-examples) section of the documentation. 
