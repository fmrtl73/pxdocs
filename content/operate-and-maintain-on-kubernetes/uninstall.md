---
title: Uninstall
weight: 2
---

Uninstalling or deleting the portworx daemonset only removes the portworx containers from the nodes. As the configurations files which PX use are persisted on the nodes the storage devices and the data volumes are still intact. These portworx volumes can be used again if the PX containers are started with the same configuration files.

### Uninstall {#uninstall}

You can uninstall Portworx from the cluster using the following steps:

1. Remove the Portworx systemd service and terminate pods by labelling nodes as below. On each node, Portworx monitors this label and will start removing itself when the label is applied.

   ```text
   kubectl label nodes --all px/enabled=remove --overwrite
   ```

2. Monitor the PX pods until all of them are terminated

   ```text
   kubectl get pods -o wide -n kube-system -l name=portworx
   ```

3. Remove all PX Kubernetes Objects

   a. If you have a copy of the spec file you used to install Portworx:

   ```text
    kubectl delete -f px-spec.yaml
   ```

   b. If you don’t, you can use the Web form:

   ```text
    VER=$(kubectl version --short | awk -Fv '/Server Version: /{print $3}')
    kubectl delete -f "https://install.portworx.com?ctl=true&kbver=$VER"
   ```

4. Remove the ‘px/enabled’ label from your nodes

   ```text
   kubectl label nodes --all px/enabled-
   ```

> **Note:**  
> During uninstall, the Portworx configuration files under `/etc/pwx/` directory are preserved, and will not be deleted.

### Delete PX Cluster configuration {#delete-px-cluster-configuration}

The commands used in this section are DISRUPTIVE and will lead to loss of all your data volumes. Proceed with CAUTION.

#### Portworx 1.3 and higher {#portworx-13-and-higher}

You can use the following command to wipe your entire Portworx cluster.

```text
curl -fsL https://install.portworx.com/px-wipe | bash
```

Above command will run a Kubernetes Job that will perform following operations:

* Detect the key value store that was being used for Portworx from the DaemonSet spec and wipe the Portworx cluster metadata from it.
* Remove Portworx systemd files from all nodes.
* Removes directories `/etc/pwx` and `/opt/pwx` from all nodes.
* Removes Portworx fingerprint data from all the storage devices used by Portworx. It also removes all the volume data from the storage devices.
* Delete all Portworx Kubernetes spec objects.

Before running the above wipe job, ensure that the Portworx spec is applied on your cluster.

#### Portworx 1.2 {#portworx-12}

You can remove PX cluster configuration by deleting the configuration files under `/etc/pwx` directory on all nodes:

* If the portworx pods are running, you can run the following command:

```text
  PX_PODS=$(kubectl get pods -n kube-system -l name=portworx | awk '/^portworx/{print $1}')
  for pod in $PX_PODS; do
      kubectl -n kube-system exec -it $pod -- rm -rf /etc/pwx/
  done
```

* otherwise, if the portworx pods are not running, you can remove PX cluster configuration by manually removing the contents of `/etc/pwx` directory on all the nodes.

> **Note**  
> If you are wiping off the cluster to re-use the nodes for installing a brand new PX cluster, make sure you use a different ClusterID in the DaemonSet spec file \(ie. `-c myUpdatedClusterID`\).

