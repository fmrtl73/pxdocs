---
title: Using Velero with Portworx
keywords: velero, disaster recovery, snapshots, persisten volumes, kubernetes, k8s, heptio ark,
description: Learn how the Portworx plugin for Velero can help with disaster recovery in your Kubernetes clusters
weight: 700
noicon: true
series: k8s-storage
aliases:
  - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/disaster-recovery
  - /portworx-install-with-kubernetes/storage-operations/disaster-recovery/
---

Velero is a utility for managing disaster recovery, specifically for your
Kubernetes cluster resources and persistent volumes. To take snapshots of
Portworx volumes through Velero you need to install and configure the Portworx
plugin.

{{<info>}}These instructions are for Velero v1.0 and higher. For older versions of Ark (previous name for the Velero project), please [click here.](ark-pre-1.0) {{</info>}}

## Install Velero Plugin

Portworx needs to be added as a plugin to your existing Velero installation.
Run the following command to install the Portworx plugin:
```text
velero plugin add portworx/velero-plugin:1.0.3
```

This should add an init container to your Velero deployment to install the plugin.

## Configure Velero to use Portworx snapshots

Once the plugin is installed, you need to create VolumeSnapshotLocation objects for Velero to use when
taking volume snapshots. These specify whether you want to take local or cloud snapshots.

Run the following command to create a VolumeSnapshotLocation for local snapshots.
```text
velero snapshot-location create portworx-local --provider portworx.io/portworx --config type=local
```

Run the following command to create a VolumeSnapshotLocation for cloud snapshots.

```text
velero snapshot-location create portworx-cloud --provider portworx.io/portworx --config type=cloud,credId=<UUID>
```

After applying the above specs you should see them when you list the VolumeSnapshotLocaions
```text
kubectl get volumesnapshotlocation -n velero
```

```output
NAME             AGE
portworx-cloud   54m
portworx-local   54m
```

The valid configuration parameters that can be given through the `--config` argument in the above commands are:

* **type**: Defines the type of snapshot. Valid values: [cloud | local]
* **credID**: Defines the credential Portworx should use when triggering a cloud snapshot.
* **PX_NAMESPACE**: Specify the namespace in which Portworx is installed. This parameter is not required if Portworx
  is installed in `kube-system` namespace. This option is similar to what is specified in other Portworx components 
  like Stork.
* **portworx.io/cloudsnap-incremental-count**: Specify the number of incremental cloud snapshots Portworx should take
  before triggering a full backup. This option is similar to what is specified in other Portworx components like Stork.

Here is a sample command that uses all the available config parameters:

```text
velero snapshot-location create portworx-cloud --provider portworx.io/portworx --config type=cloud,credId=<UUID>,PX_NAMESPACE=portworx,portworx.io/cloudsnap-incremental-count=7
```

## Creating backups

Once you have installed and configured the plugin, every time you take backups
using Velero and include PVCs, it will also take Portworx snapshots of your volumes.

### Local Backups

To backup all your apps in the default namespace and also create local snapshots
of the volumes, you would use `portworx-local` for the snapshot location:

```text
velero backup create default-ns-local-backup --include-namespaces=default --snapshot-volumes \
     --volume-snapshot-locations portworx-local
```

```output
Backup request "default-ns-local-backup" submitted successfully.
Run `velero backup describe default-ns-local-backup` for more details.
```

### Cloud Backups

To backup all your apps in the default namespace and also create cloud backups
of the volumes, you would use `portworx-cloud` for the snapshot location:

```text
velero backup create default-ns-cloud-backup --include-namespaces=default --snapshot-volumes \
     --volume-snapshot-locations portworx-cloud
```

```output
Backup request "default-ns-cloud-backup" submitted successfully.
Run `velero backup describe default-ns-cloud-backup` for more details.
```

## Listing backups

Once the specs and volumes have been backed up you should see the backup marked
as `Completed` in velero.

```text
velero get backup
```

```output
NAME                      STATUS      CREATED                         EXPIRES   STORAGE LOCATION    SELECTOR
default-ns-local-backup   Completed   2018-11-11 20:10:45 +0000 UTC   29d       default             <none>
default-ns-cloud-backup   Completed   2018-11-11 20:15:45 +0000 UTC   29d       default             <none>
```

## Restoring from backups

When restoring from backups, a clone volume will be created from the snapshot and
bound to the restored PVC. To restore from the backup created above you can run
the following command:

```text
velero restore create --from-backup default-ns-local-backup
```

```output
Restore request "default-ns-local-backup-20181111201245" submitted successfully.
Run `velero restore describe default-ns-local-backup-20181111201245` for more details.
```
