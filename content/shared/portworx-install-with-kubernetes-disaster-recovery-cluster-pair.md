---
hidden: true
---

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
