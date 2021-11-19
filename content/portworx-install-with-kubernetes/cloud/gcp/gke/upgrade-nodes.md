---
title: Upgrade GKE node types
weight: 4
keywords: GKE, Google Kubernetes Engine, k8s, gcloud
description: 
noicon: true
---

If you're running Portworx on GKE and want to migrate to nodes with more resources, you can do this by adding new nodes and retiring old ones. Perform the following steps to migrate nodes:

{{<info>}}
**NOTE:** This upgrade method only works if you're using an internal KVDB.
{{</info>}}

1. [Add a new node pool](https://cloud.google.com/kubernetes-engine/docs/how-to/node-pools#add) containing your desired node type.

2. Ensure all nodes in the new node pool have Portworx in storageless mode.

3. [Disable auto scaling](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-autoscaler#disable_autoscaling) in the old pool.

4. Choose a node from your old node pool you want to remove:

    ```text
    pxctl cluster list
    ```
    ```output
    Cluster ID: 8ed1d365-fd1b-11e6-b01d-0242ac110002
    Status: OK

    Nodes in the cluster:
    ID					DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION		STATUS
    bf9eb27d-415e-41f0-8c0d-4782959264bc	147.75.99.243	0.125078	34 GB		33 GB		N/A		1.1.4-6b35842	Online
    7d97f9ea-a4ff-4969-9ee8-de2699fa39b4	147.75.99.171	0.187617	34 GB		33 GB		N/A		1.1.4-6b35842	Online
    492596eb-94f3-4422-8cb8-bc72878d4be5	147.75.99.189	0.125078	34 GB		33 GB		N/A		1.1.4-6b35842	Online
    ```

5. Once you've chosen a node ID, enter the following `kubectl` command to drain the node, specify your own <node-id>: 
    
    ```text
    kubectl drain <node-id> --ignore-daemonsets --delete-emptydir-data
    ```

6. Drain volume attachments:

    ```text
    pxctl service node drain-attachments submit --node <node-id>
    ```

7. Delete the node from Kubernetes:
    
    ```text
    kubectl delete node <node>
    ```

8. In the GKE console, find the node by name and [delete the instance](https://cloud.google.com/compute/docs/instances/deleting-instance#delete_an_instance). 

9. Choose a storageless node and restart Portworx so it becomes a storage node:

    ```text
    kubectl label <node> px/service=restart --overwrite
    ```

    Wait until the storage node is back up before proceeding.

10. Re-enable attachments on the node:
    
    ```text
    pxctl service node uncordon-attachments --node <node-id>
    ```

Repeat steps 5 to 10 for each remaining node in your cluster. 