---
title: Incremental cloud snapshots
weight: 700
keywords: snapshots, cloudsnaps, kubernetes, k8s, incremental
description: Learn how to configure the interval for full snapshots in cloudsnaps.
---

When Portworx takes and uploads backups for cloud snapshots (cloudsnaps), it first takes and uploads a full backup to your cloud provider. After the initial upload, Portworx uploads a series of incremental backups punctuated by an occasional full backup. By default, Portworx uploads a full backup every 7th cloud backup it performs. You can control the frequency with which Portworx uploads a full backup by configuring cluster options. 

{{<info>}}
**NOTE:** This is a cluster-wide setting.
{{</info>}}

## View full backup frequency

You can view the currently set frequency by searching for it in the `pxctl cluster options list` output:
 
```text
pxctl cluster options list | grep frequency
```
```output
Cloudsnap full backup frequency                         : 7
```
 
## Change full backup frequency

To modify the frequency with which Portworx uploads full backups for cloudsnaps, enter the following `pxctl cluster options update` command with the `--cloudsnap-full-backup-frequency` flag and a valid frequency range. The following example updates the frequency to every 4 cloudsnaps:

```text
pxctl cluster options update --cloudsnap-full-backup-frequency 4
```

{{<info>}}
**NOTE:** If you set the `--cloudsnap-full-backup-frequency` to a very high value, Portworx may take longer to restore your cloudsnaps, since there's a large number of incremental snapshot data to piece together before reaching a full backup. Additionally, these cloudsnaps may be larger, as they hold more incremental snapshots. 

If you set the frequency to a very low value, Portworx will send full backups more often.
{{</info>}}