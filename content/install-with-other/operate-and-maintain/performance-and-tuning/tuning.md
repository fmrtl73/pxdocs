---
title: Performance Tuning
keywords: performance tuning, container level optimization, volume level optimization, Portworx Enterprise,
description: Create, manage and inspect storage volumes with pxctl CLI. Discover how to use Docker together with Portworx!
weight: 1
---

Portworx has best practices for both global container level optimization, as well as volume granular optimization.

## Global performance tuning

As of Portworx version 1.3, it is recommended to use a journal device to absorb Portworx metadata writes. Journal writes are small with frequent syncs and therefore only SSD/NVME should be configured as a journal device.

In 1.x, the journal device should be 2GB, and in 2.x it should be 3GB. Using a larger device will not help, since Portworx  will only use these amounts of storage. The journal device can be specified via the `-j` option during installation, documented [here](/install-with-other/docker/standalone).

{{<info>}}
You **must** ensure that the journal device is faster than your storage device allocated for Portworx. If the journal device is slower than the actual storage drive, your overall performance will be lower and match the lower of two devices.
{{</info>}}

{{<info>}}
Cloud providers match the drive's performance based on it's size.  So if you select a small sized journal device, your performance will be worse.  For a cloud drive, provide a partition from the larger storage drive as your journal device.
{{</info>}}

{{<info>}}
As of Portworx 1.4, {{<companyName>}} recommends using the `-j auto` option.  This allows Portworx to create its own journal partition on the best drive.
{{</info>}}

If you are upgrading to 1.3 and want to add a journal device to an existing node, follow [these instructions](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/add-journal-dev).

## Tune volume performance with pxctl

Portworx optimizes performance for specific application access patterns using IO profiles. You can set these IO profiles by providing the `io_profile` option while creating the volume. For example:

```text
pxctl volume create --size=10 --repl=3 --io_profile=db_remote demovolume
```

or

```text
docker volume create -d pxd io_priority=high,size=10G,repl=3,io_profile=random,name=demovolume
```

For more information about IO profiles, see the [IO profiles](/concepts/io-profiles) section of the documentation. 