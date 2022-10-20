---
title: 2. Pair Clusters
weight: 200
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: Find out how to pair your clusters
aliases:
    - /portworx-install-with-kubernetes/disaster-recovery/px-metro/2-pair-clusters/
---
## Understand cluster pairing

In order to failover an application running on one Kubernetes cluster to another Kubernetes cluster, you need to migrate the resources between them.
On Kubernetes you will define a trust object required to communicate with the other Kubernetes cluster called a ClusterPair. This creates a pairing between the scheduler (Kubernetes) so that all the Kubernetes resources can be migrated between them.
Throughout this section, the notion of source and destination clusters apply only at the Kubernetes level and does not apply to Storage, as you have a single Portworx storage fabric running on both the clusters.
As Portworx is stretched across them, the volumes do not need to be migrated.

For reference:

* **Source Cluster** is the Kubernetes cluster where your applications are running.
* **Destination Cluster** is the Kubernetes cluster where the applications will be failed over, in case of a disaster in the source cluster.

## Generate and Apply a ClusterPair Spec

In Kubernetes, you must define a trust object called **ClusterPair**. Portworx requires this object to communicate with the destination cluster. The ClusterPair object pairs the Portworx storage driver with the Kubernetes scheduler, allowing the volumes and resources to be migrated between clusters.

The ClusterPair is generated and used in the following way:

   * The **ClusterPair** spec is generated on the **destination** cluster.
   * The generated spec is then applied on the **source** cluster

Perform the following steps to create a cluster pair:

{{<info>}}
**NOTE:** You must run the `pxctl` commands in this document either on your Portworx nodes directly, or from inside the Portworx containers on your master Kubernetes node. 
{{</info>}}

### Generate a ClusterPair on the destination cluster

To generate the **ClusterPair** spec, run the following command on the **destination** cluster:

```text
storkctl generate clusterpair -n migrationnamespace remotecluster
```
Here, the name (remotecluster) is the Kubernetes object that will be created on the **source** cluster representing the pair relationship.

During the actual migration, you will reference this name to identify the destination of your migration.

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
    creationTimestamp: null
    name: remotecluster
    namespace: migrationnamespace
spec:
   config:
      clusters:
         kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            certificate-authority-data: <CA_DATA>
            server: https://192.168.56.74:6443
      contexts:
         kubernetes-admin@kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            cluster: kubernetes
            user: kubernetes-admin
      current-context: kubernetes-admin@kubernetes
      preferences: {}
      users:
         kubernetes-admin:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            client-certificate-data: <CLIENT_CERT_DATA>
            client-key-data: <CLIENT_KEY_DATA>
    options:
       <insert_storage_options_here>: ""
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

Save the resulting spec to a file named `clusterpair.yaml`.

### Using Rancher Projects with ClusterPair

{{<info>}}**NOTE**: If you are not using Rancher, skip to the [next section](#apply-the-generated-clusterpair-on-the-source-cluster). {{</info>}}

Rancher has a concept of Projects that allow grouping of resources and Kubernetes namespaces. Depending on the resource and how it is created, Rancher adds the following label or annotation:
```text
field.cattle.io/projectID: <project-short-UUID>
```
The `projectID` uniquely identifies the project, and the annotation or label on the Kubernetes object provides a way to tie a Kubernetes object back to a Rancher project. 

From version 2.11.2 or newer, Stork has the capability to map projects from the source cluster to the destination cluster when it migrates Kubernetes resources. It will ensure that the following are transformed
when migrating Kubernetes resources to a destination cluster:
* Labels and annotations for projectID `field.cattle.io/projectID` on any Kubernetes resource on the source cluster are transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a NetworkPolicy object which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a Pod object (Kubernetes version 1.24 or newer) which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.

{{<info>}}
**NOTE:**

* Rancher project mappings are supported only with Stork version 2.11.2 or newer.
* All the Rancher projects need to be created on both the source and the destination cluster.
{{</info>}}

While creating the ClusterPair, use the argument `--project-mappings` to indicate which projectID on the source cluster maps to a projectID on the destination cluster. 
For example:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster> --project-mappings  <projectID-A1>=<projectID-A2>,<projectID-B1>: <projectID-B2>
```
The project mappings are provided as a comma-separate key=value pairs. In this example, `projectID-A1` on source cluster maps to `projectID-A2` on the destination cluster, while `projectID-B1` on the source cluster maps to `projectID-B2`
on the destination cluster.

### Apply the generated ClusterPair on the source cluster

On the **source** cluster create the clusterpair by applying the generated spec.

```text
kubectl create -f clusterpair.yaml
```

### Verify the Pair status
Once you apply the above spec on the source cluster you should be able to check the status of the pairing using storkctl on the source cluster.

```text
storkctl get clusterpair
```

```output
NAME               STORAGE-STATUS   SCHEDULER-STATUS   CREATED
remotecluster      NotProvided      Ready              09 Apr 19 18:16 PDT
```

So, on a successful pairing you should see the "Scheduler Status" as "Ready" and the "Storage Status" as "Not Provided"

Once the pairing is configured, applications can now failover from one cluster to another. In order to achieve that, we need to migrate the Kubernetes resources to the destination cluster. The next step will help your synchronize the Kubernetes resources between your clusters.
