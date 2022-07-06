---
title: Configure Pure Storage FlashArray as a Direct Access volume
weight: 
linkTitle: Pure FlashArray PVCs
keywords: flasharray, direct access, volume, pvc, storage
description: Learn how to set up Pure FlashArray as a direct access volume.
---

On-premises users who want to use Pure Storage FlashArray with Portworx on Kubernetes can attach FlashArray as a Direct Access volume. Used in this way, Portworx directly provisions FlashArray volumes, maps them to a user PVC, and mounts them to pods. Once mounted, the application writes data directly onto FlashArray. As a result, this mounting method doesn’t use storage pools.

FlashArray Direct Access volumes support the following CSI operations:

* Basic filesystem operations: create, mount, expand, clone, unmount, delete
* Mount options: Configure file system mount options
* Snapshots
* Quality of service (QoS) settings (requires at least one FlashArray with Purity version 5.3.0 or newer and REST API version 1.17 or newer)

{{<info>}}**NOTE:** FlashArray Direct Access volumes have the following limitations:

* RawBlock, NVME-RoCE, volume import, and CSI ephemeral volumes are not supported.
* `CreateOperations` is not honored by Portworx.
{{</info>}}

## Install Portworx and configure FlashArray
Before you install Portworx, ensure that you meet the prerequisites. You must provide Portworx with your FlashArray configuration details during installation.

### Prerequisites

* Have a Pure Storage FlashArray with Purity version 5.3.0 or newer
* Use the FC or iSCSI protocol
* Create a Pure secret `px-pure-secret` under the `portworx` namespace before installing Portworx
* Install Portworx version 2.11.0 or newer
* Use the Portworx Operator 1.8.1 or newer
* Enable CSI for Portworx
* Have the latest Linux multipath software package for your operating system
    * FlashArray direct access volumes do not support user friendly names in multipath devices. If `user_friendly_names` is present in your `multipath.conf` file, it must be set to `no`.
    * If you have a vSphere Cloud environment, add the following in your `multipath.conf` file:
        ```text
        blacklist {
            devnode "vda"
            device {
                vendor "VMware"
            }
        }
        ```
* Have the latest Filesystem utilities/drivers
* Have the latest external array management library package `libstoragemgt_udev`
    * Red Hat and CentOS only: Ensure that the second action - `CAPACITY_DATA_HAS_CHANGED` - is uncommented and you have restarted the `udev` service
* Have the latest iSCSI initiator software for your operating system (Optional; required for iSCSI connectivity)
* Have the latest FC initiator software for your operating system (Optional; required for FC connectivity)

To use the CSI snapshot feature, you also need to install the following:

* [Snapshot V1 CRDs](https://github.com/kubernetes-csi/external-snapshotter/tree/master/client/config/crd)
* [Snapshot controller](https://kubernetes-csi.github.io/docs/snapshot-controller.html)

    * You can also install the snapshot controller by adding the following lines to your StorageCluster:

      ```text
        csi:
          enabled: true
          installSnapshotController: true
      ```

## Deploy Portworx

Once you've ensured you meet the prerequisites and your physical network topology is appropriately configured, you're ready to deploy Portworx.

1. Create a JSON file named `pure.json` that contains your FlashArray information:

    ```text
    {
        "FlashArrays": [
            {
                "MgmtEndPoint": "<fa-management-endpoint>",
                "APIToken": "<fa-api-token>"
            },
        ]
    }
    ```

    {{<info>}}
**NOTE:** You can add FlashBlade configuration information to this file if you're configuring both FlashArray and FlashBlade together. Refer to the [JSON file](/reference/pure-reference/pure-json-reference/) reference for more information.
    {{</info>}}

2. Enter the following `kubectl create` command to create a Kubernetes secret called `px-pure-secret`:
    
    ```text
    kubectl create secret generic px-pure-secret --namespace kube-system --from-file=pure.json=<file path>
    ```
    ```output
    secret/px-pure-secret created
    ```
   
    {{<info>}}
**NOTE:** You must name the secret `px-pure-secret`.
    {{</info>}}


3. Deploy Portworx [on your on-premises Kubernetes cluster](/portworx-install-with-kubernetes/on-premise/other) or [in your cloud environment](/install-portworx/cloud/). Ensure that **CSI** is enabled.

Once deployed, Portworx detects that the FlashArray secret is present when it starts up, and can use the specified FlashArray as a Direct Access volume.

## Use FlashArray as a Direct Access volume

Once you’ve configured Portworx to work with your FlashArray, you can create a StorageClass and reference it in any PVCs you create.

### Create a StorageClass

Create a StorageClass spec and set `parameters.backend` to `"pure_block"`. Here is an example StorageClass spec:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-portworx-fa-direct-access
provisioner: pxd.portworx.com
parameters:
  backend: "pure_block"
  max_iops: "1000"
  max_bandwidth: "1G"
allowVolumeExpansion: true
```

### Create a PVC

Create a PersistentVolumeClaim and reference the StorageClass you created by entering the name that you gave your StorageClass in the `spec.storageClassName` field. For example:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pure-claim-block
  labels:
    app: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: sc-portworx-fa-direct-access
```

### Mount to a pod

Create a Pod and reference the PVC you created by entering the name you gave your PVC in the `persistentVolumeClaim.claimName` field. For example:

```text
kind: Pod
apiVersion: v1
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  volumes:
  - name: pure-vol
    persistentVolumeClaim:
      claimName: pure-claim-block
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: pure-vol
      mountPath: /data
    ports:
    - containerPort: 80
```

Once you apply the Pod, you can use `watch kubectl get pods` to look for a `STATUS` of `Running`. Once the pod is running, you can see it as a connected host for your volume.

### Clone a PVC

To clone PVC, create a PersistentVolumeClaim with `dataSource.kind` set to `PVC` and `dataSource.name` set to the name of the PVC you wish to clone. For example:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-clone
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: sc-portworx-fa-direct-access
  dataSource:
    kind: PersistentVolumeClaim
    name: pure-claim-block
    apiGroup: snapshot.storage.k8s.io
```

### Take a snapshot

To take a snapshot, perform the following steps:

1. Create a SnapshotClass with `driver` set to `pdx.portworx.com`. For example:

	```text
	kind: VolumeSnapshotClass
	apiVersion: snapshot.storage.k8s.io/v1
	metadata:
	   name: px-fa-direct-access-snapshotclass
	   annotations:
	      snapshot.storage.kubernetes.io/is-default-class: "true"
	driver: pxd.portworx.com
	deletionPolicy: Delete
	```

2. Create a VolumeSnapshot where `volumeSnapshotClassName` is set to the name you gave your VolumeSnapshotClass, and `source.persistentVolumeClaimName` is set to the name you gave your volume. For example:

	```text
	kind: VolumeSnapshot
	apiVersion: snapshot.storage.k8s.io/v1beta1
	metadata:
	  name: volumesnapshot-of-pure-claim-block
	spec:
	  volumeSnapshotClassName: px-fa-direct-access-snapshotclass
	  source:
	    persistentVolumeClaimName: pure-claim-block
	```

Once you have applied the VolumeSnapshot to create a snapshot, you can view the snapshot with `kubectl get volumesnapshot` or in the array UI under **Volume Snapshots**, where it has the same functions as other types of Pure snapshots.

{{<info>}}
**NOTE:** If you do not see your VolumeSnapshot when you run `kubectl get volumesnapshot`, run `kubectl get crd` to get the full path of the CRD, then use that full path in your `kubectl get` command. For example:

```text
kubectl get volumesnapshots.snapshot.storage.k8s.io
```
{{</info>}}

### Restore a snapshot

To restore a snapshot to a new PVC, create a PersistentVolumeClaim with `dataSource.kind` set to `VolumeSnapshot` and `dataSource.name` set to the name of your VolumeSnapshot. For example:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-restore
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: sc-portworx-fa-direct-access
  dataSource:
    kind: VolumeSnapshot
    name: volumesnapshot-of-pure-claim-block
    apiGroup: snapshot.storage.k8s.io
```

Once you have applied the PersistentVolumeClaim to create a PVC, you can view it with `kubectl get pvc` or in the array UI. The array UI shows under **Details** that the source of the new PVC is the PVC from which the snapshot was made.

### Expand a PVC

To expand a PVC of a FlashArray direct access volume, run the command `kubectl edit pvc <pvcName>`, then change the size in the spec.

### Delete a PVC

To delete a PVC of a FlashArray direct access volume, use the following command:

```text
kubectl delete pvc <pvcName>
```


## Related topics

* [CSI topology for FlashArray and FlashBlade Direct Access volumes](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/cluster-topology/csi-topology)