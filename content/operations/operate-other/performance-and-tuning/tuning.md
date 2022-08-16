---
title: Performance Tuning
keywords: performance tuning, container level optimization, volume level optimization, Portworx Enterprise,
description: Create, manage and inspect storage volumes with pxctl CLI. Discover how to use Docker together with Portworx!
weight: 100
aliases:
    - /install-with-other/operate-and-maintain/performance-and-tuning/tuning/
---
Portworx has best practices for both global container level optimization, as well as volume granular optimization.

## Global performance tuning

{{<companyName>}} recommends you use a journal device to absorb Portworx metadata writes. Journal writes are small with frequent syncs and therefore only SSD/NVME should be configured as a journal device.

The journal device should be 3GB. Using a larger device will not help, since Portworx will only use these amounts of storage. You can specify the journal device using the `-j` option during installation. See the [Install on Docker Standalone](/install-portworx/install-with-other/docker/standalone) page for more details.

{{<info>}}
**NOTE:**

- You **must** ensure that the journal device is at least as fast as the fastest storage device on the node allocated for Portworx. If the journal device is slower than the actual storage drive, your overall performance will decrease to match the lower of two the devices.
- Cloud providers match the drive's performance based on its size. So if you select a small sized journal device, your performance will be worse.  For a cloud drive, provide a partition from the larger storage drive as your journal device.
- {{<companyName>}} recommends using the `-j auto` option. This allows Portworx to create its own journal partition on the best drive.
{{</info>}}

## Tune volume performance with pxctl

Portworx optimizes performance for specific application access patterns using IO profiles. You can set these IO profiles by providing the `io_profile` option while creating the volume. For example:

```text
pxctl volume create --size=10 --repl=3 --io_profile=db_remote demovolume
```

or

```text
docker volume create -d pxd io_priority=high,size=10G,repl=3,io_profile=db_remote,name=demovolume
```

## Related topics

* [IO profiles](/concepts/io-profiles/)
* [Performance optimization options in the IO path](/reference/performance-io-path/)
