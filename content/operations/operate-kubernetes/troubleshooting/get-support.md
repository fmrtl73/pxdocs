---
title: Troubleshoot Portworx on Kubernetes
linkTitle: Troubleshoot and Get Support
weight: 300
keywords: Troubleshoot, debug, kubernetes, k8s, collecting logs
description: Useful information for troubleshooting Portworx on Kubernetes
series: support
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/troubleshooting/troubleshoot-and-get-support/
    - /operations/operate-kubernetes/troubleshooting/troubleshoot-and-get-support/
---

## Troubleshoot problems

The following sections provide troubleshooting tips for common problem areas:

### Portworx Node is down

* `ssh` into your cluster node that has `kubectl` installed with your `kubeconfig` and check the Kubernetes cluster status using `kubectl` to ensure cluster nodes are in the `Ready` status:
    ```text
    kubectl get node -o wide
    ```
* If a node is not ready, describe that node to see why and take corrective action:
    ```text
    kubectl describe node <nodename>
    ```
* If the previous command does not help identify the problem, log in as root and consider running the `journalctl` command on the node in question to identify the problem:
    ```text
    journalctl -u kubelet
    ```
* If the Kubernetes cluster is healthy, check Portworx alerts using `pxctl` from the node, either through `ssh` or using `kubectl exec`. Alerts may help you understand why the Portworx node is down:
    ```text
    pxctl alerts show
    ```
* You can also enter the `pxctl status` command to check the status on the respective node where portworx is running:
    ```text
    pxctl status
    ```
* If you find no useful information in the `pxctl status` output, check your Portworx pods to confirm they are up and running:
    
    ```text
    kubectl get pods -n <name-space> -l name=portworx
    ```

    * If necessary, describe the respective Portworx pod to identify the problem:
        ```text
        kubectl describe pods <px-podname> -n <name-space>
        ```

    * If necessary, check the `journalctl` logs from the node in question to further help identify the problem:
        ```text
        journalctl -lfu portworx*
        ```     

* Check all Portworx pods running in kube-system or other namespace and confirm they are up and running:
    ```text
    kubectl get pods -n <name-space>
    ```
* Describe the respective pod running in kube-system or other namespace to identify the problem.
    ```text
    kubectl describe pod <podname> -n <name-space>
    ```

### Portworx logs reports "Node is not in quorum", kvdb error: "context deadline exceeded"

* `ssh` into the respective nodes and run `pxctl status` on each node to check the Portworx cluter status:
    ```text
    pxctl status
    ```
* If running internal KVDB check KVDB cluster members and confirm the health status using pxctl:
    ```text
    pxctl service kvdb members
    ```
* If quorum has been lost perform the following before contacting technical support:
    * Save px-diags on each affected node (captures all logs)  
      ```text
      pxctl service diags -a
      ```
    * Make backups of your config map for px-bootstrap and px-cloud-drive
      ```text
      kubectl get cm -n kube-system | grep px
      ``` 
      ```text
      kubectl get cm <px-bootstrap> -n kube-system -o yaml > px-bootstrapbkp.yaml
      ``` 
      ```text
      kubectl get cm <px-cloud-drive> -n kube-system -o yaml > px-cloud-drivebkp.yaml
      ``` 
    * Collect KVDB end points using pxctl:
      ```text
      pxctl service kvdb endpoints
      ```       
    * Contact technical support (see below)
* If using external etcd, check your external etcd cluster status.
    * Portworx container will fail to come up if it cannot reach etcd. For etcd installation instructions refer this [doc](/operations/operate-kubernetes/etcd).
        * The etcd location specified when creating the Portworx cluster needs to be reachable from all nodes.
        * For external Etcd run `curl <etcd_location>/version` from each node to ensure reachability. For e.g `curl "http://192.168.33.10:2379/version"`
    * If you deployed etcd as a Kubernetes service, use the ClusterIP instead of the kube-dns name. Portworx nodes cannot resolve kube-dns entries since Portworx containers are in the host network.

### Portworx pxctl cluster summary reports Status "Online", StorageStatus "(StorageDown)" "Full or Offline"

1. Identify the node and the storage pool in question by running pxctl (ssh into the respective node) status:
    ```text
    pxctl status
    ```
2. From the same node, inspect the pool to identify the disk device that makes up the pool:
    ```text
    pxctl service pool show
    ```
3. Logged in as root, identify why the disk is failing by running dmesg
    ```text
    dmesg | grep error
    ```

To correct the problem: 

* Remove or replace the drive following these instructions:
  [Remove or replace](/reference/cli/remove-failed-drive/)

* If the pool is full follow these instructions:
  [Expand your storage pool size](/operations/operate-kubernetes/storage-operations/create-pvcs/expand-storage-pool/)

### Performance related

* Run Grafana dashboard to identify volumes, pools, nodes, network and other components. 
    * [Grafana](/operations/operate-other/monitoring/grafana/)
    * [Dashboard](/samples/k8s/pxc/portworx-performance-dashboard.json)
    * Requires the latest charts from Portworx release 2.10.0 and up.
* Refer to the following performance tuning document:
[Tune Performance](/operations/operate-kubernetes/tune-performance/)
* There are many performance tuning enhancements in the latest release of Portworx. Please see:
[Portworx release notes](/release-notes/portworx/)

### PVC Controller pod failed to start

If you are running Portworx in managed Kubernetes service provider and run into port conflict in the PVC controller, you can overwrite the default PVC Controller ports using the `portworx.io/pvc-controller-port` and `portworx.io/pvc-controller-secure-port` annotations on the `StorageCluster` object:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: portworx
  namespace: kube-system
  annotations:
    portworx.io/pvc-controller-port: "10261"
    portworx.io/pvc-controller-secure-port: "10262"
...
```

## Collect Portworx logs

Run the following command on the suspect or affected nodes running Portworx:
```text
pxctl service diags -a
```
Note: 
Include these logs when contacting Portworx support, along with generated diags located in `/var/cores/<node-x-x-diags>-<timestamp>.tar.gz`

## Generate stack traces

Portworx support will occasionally request stack traces to help you troubleshoot. Enter the following command on the troubled node to create a `*.stack` file in the `/var/cores` directory with the latest timestamp:

```text
pxctl service diags --profile
```
    
## Contact support

View your options for contacting support by visiting the Portworx support page:

{{< widelink url="/support/" >}}Portworx support{{</widelink>}}
