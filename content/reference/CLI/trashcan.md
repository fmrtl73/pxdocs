---
title: Volume trash can
description: Explanation of the trash can feature
keywords: volume, delete, trashcan, trash, recycle, undelete, restore
weight: 1
---

The trash can feature provides protection against accidental or inadvertent volume deletions which could result in loss of data. In a clustered environment such as Kubernetes, unintended deletion of a PV or a namespace will cause volumes to be lost. This feature is recommended in any environment which is prone to such inadvertent deletions, as it can help to prevent data loss.

## Enable the trash can

To enable the trash can feature on a cluster, run the following command:

```text
pxctl cluster options update --volume-expiration-minutes <minutes>
```

`<minutes>` is an integer that represents the number of minutes a volume snapshot is retained after you delete a volume.

## Check trash can settings

To check a cluster's options to see if the trash can is enabled, and what the retention period is, run the following command:

```text
pxctl cluster options list | grep expiration
```
```output
Volume expiration minutes                               : 0s
```

## Restore a volume

When the trash can is enabled, you can list volumes in the trash can with the following command:

```text
pxctl volume list --trashcan
```
```output
DELETE TIME                   ID                  NAME                  SIZE   HA  SHARED  ENCRYPTED  IO_PRIORITY  STATUS         DELETE_TIMER
Tue Mar 29 21:51:16 UTC 2022  780196670250220779  newvol-tc-1648590676  1 GiB  1   no      no         LOW          up - detached  9m33s
```
In the sample output above, the trash can volume name is in the format `<original-vol-name>-tc-<original-vol-id>`.

To restore a volume, use the following command:

```text
pxctl volume restore --trashcan <trashcan-vol-id> <desired-vol-name>
```
```output
Successfully restored: <desired-vol-name> from <trashcan-vol-id>
```

This removes the volume from the trash can as well.


## Disable the trash can

To disable the trash can feature on a cluster, set `volume-expiration-minutes` equal to `0`:

```text
pxctl cluster options update --volume-expiration-minutes 0
```
