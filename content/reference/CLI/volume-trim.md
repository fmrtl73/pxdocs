---
title: Maintain volumes using Filesystem Trim
linkTitle: Volume trim
keywords: portworx, pxctl, command-line tool, cli, reference, volume trim, discard
description: Maintain volumes using filesystem trim
weight: 17
---

A typical Portworx volume is formatted with either ext4 or xfs and then used by a container application to store its content files and directories. Over time, your application might create and delete files and directories. On the volume, the space which was previous used by a deleted file gets freed in the filesystem metadata and the underlying block device is unaware of this fact. This can lead to the following inefficiencies:

* On thin provisioned volumes, the freed space in the volume does not translate into free space in the pool. This means that other volumes in the pool that require space might not be able to get it from the pool.
* On SSDs, the block device performs better when it has knowledge of all the freed blocks that the user no longer requires. This information is used by the SSD firmware to perform wear-leveling more efficiently to improve the service life of the storage device and also provide better I/O performance. When the information about the blocks freed in the filesystem is not available to the block device, it creates hot spots in the device that cause it to wear more than rest of the blocks in the device.

To address these inefficiencies, you can instruct the filesystem to inform the block device of all the unused blocks which were previously used by issuing a `FITRIM` `ioctl` command to the mounted filesystem. The filesystem in turn issues a `DISCARD` request for the freed blocks to the block device.

You can use automatic filesystem trim operations, or you can perform filesystem trim operations manually.

## Automatic filesystem trim operations

You can enable automatic filesystem trimming (auto fstrim) at the volume, node, or cluster level. When all of the following conditions are met, auto fstrim monitors the unused space in all filesystems mounted on Portworx volumes and automatically triggers a trim job to return unused space back to the pool, and you do not have to manually issue trim jobs:

* Volumes have `nodiscard` enabled
* Auto fstrim is enabled at the cluster level _or_ on the node where the volume is attached

Auto fstrim takes into account current workloads in the system and dynamically adjusts the rate at which it performs the trim job. This prioritizes user application performance while also optimizing the trim rate when application load is low.


{{<info>}}
**NOTE:**

* Enabling auto fstrim or changing IO rates has a small delay before taking effect.
* If a volume is unmounted or detatched, Portworx automatically stops auto fstrim on that volume. If the volume is remounted, auto fstrim automatically starts again.
{{</info>}}

### Enable auto fstrim

#### Enable on a new volume

To create a new volume with auto fstrim enabled, specify the following options at volume creation:

```text
pxctl volume create <volume_name> --nodiscard
```
```output
Volume successfully created: <volume_ID>
```

You can verify that the volume has auto fstrim enabled with the following command:

```text
pxctl volume inspect <volume_name>
```
```output
...
Mount Options                   :   nodiscard
...
Auto Fstrim                     :   true
...
```


#### Enable on an existing volume

To enable auto fstrim on an existing volume, run the following command:

```text
pxctl volume update --nodiscard on <volume_name>
```

You can verify that the volume has auto fstrim enabled with the following command:

```text
pxctl volume inspect <volume_name>
```
```output
...
Mount Options                   :   nodiscard
...
Auto Fstrim                     :   true
...
```

#### Enable on a node

To enable auto fstrim for a node, run the following command:

```text
pxctl cluster options update --runtime-options-action update-node-specific --runtime-options-selector node=<node_uuid> --runtime-options NodeAutoFstrimEnabled=1
```

{{<info>}}
**Note:** Running this command will overwrite any auto fstrim IO rates you have set at the node level to their default values.
{{</info>}}


#### Enable on a cluster

To enable auto fstrim for a cluster, run the following command:

```text
pxctl cluster options update --auto-fstrim on
```

### IO rates

Auto fstrim allows you to choose specific IO minimum and maximum rates at the cluster or node level.


#### View existing IO rates

##### View rates at the cluster level

To view your existing IO rates at the cluster level, use the following command:

```text
pxctl cluster options list | grep -i fstrim
```
```output
Auto Fstrim                : <on or off>
Max Fstrim IO rate         : <rate>
Min Fstrim IO rate         : <rate>
```

##### View rates at the node level

To view your existing IO rates at the node level, use the following command:

```text
pxctl cluster options list | grep Runtime
```

If you have manually set both rates, you will see output similar to the following:
```output
Runtime options                                         : selector: <node_uuid>, options: NodeFstrimMaxIoRate=<rate>,NodeFstrimMinIoRate=<rate>
```

If the rates are both set to the default, and you have enabled auto fstrim on the node, you will see output similar to the following:

```output
Runtime options                                         : selector: node=<node_uuid>, options: NodeAutoFstrimEnabled=1
```

#### Change IO rates

##### Change rates at the cluster level

To change your minimum IO rate at the cluster level, run the following command:

```text
pxctl cluster options update --fstrim-min-io-ratio <rate>
```
```output
Successfully updated cluster-wide options
```

To change your maximum IO rate at the cluster level, run the following command:

```text
pxctl cluster options update --fstrim-max-io-ratio <rate>
```
```output
Successfully updated cluster-wide options
```

##### Change rates at the node level

To change  your minimum and maximum IO rates at the node level, run the following command:

```text
pxctl cluster options update --runtime-options-action update-node-specific --runtime-options-selector node=<node_uuid> --runtime-options NodeFstrimMaxIoRate=<rate>,NodeFstrimMinIoRate=<rate>
```
```output
Successfully updated cluster-wide options
```


{{<info>}}
**Note:**

If you issue the previous command with only `NodeFstrimMaxIoRate` or `NodeFstrimMinIoRate` defined, not both, the unspecified value will be set to the default value, and any previous change will be overwritten.

{{</info>}}

##### Disable node-level rates

To disable node-specific IO rate options but keep auto fstrim enabled, use the following command:

```text
pxctl cluster options update --runtime-options-action update-node-specific --runtime-options-selector node=<node_uuid> --runtime-options NodeAutoFstrimEnabled=1
```
```output
Successfully updated cluster-wide options
```

### View auto fstrim usage

To view usage information about locally attached volumes that are auto fstrim eligible, run the following command:

```text
pxctl volume af usage
```

For information about running auto fstrim processes, use the following command:

```text
pxctl volume af status
```

This can have a variety of outputs if the process is not running, such as:
```output
AutoFsTrimStatus: No auto fstrim volumes found
```
```output
AutoFsTrimStatus: Filesystem Trim busy, please retry
```
```output
Auto fs trim is not running for any volume.
```

When auto fstrim is running, this command displays output with the following columns of information:
```output
Volume ID    Status    Volume Size    Trimmable Space    Trimmed Space    Average Rate    Current Rate    Percentage Complete
```

{{<info>}}
**NOTE:** Auto fstrim needs to run for a short time before the average rate displays.
{{</info>}}

### View auto fstrim status for a volume

You can also specify a volume name to view its auto fstrim status:

For information about running auto fstrim processes, use the following command:

```text
pxctl volume af status <volume_name>
```

Additionally, you can specify a flag to have these results returned in JSON format:

```text
pxctl volume af status <volume_name> -j
```

### Disable auto fstrim

#### Disable on a volume

To turn off auto fstrim for a volume, run the following command:

```text
pxctl volume update <volume_name> --nodiscard off
```

#### Disable on a node

To turn off auto fstrim for a node, run the following command:

```text
pxctl cluster options update --runtime-options-action update-node-specific --runtime-options-selector node=<node uuid> --runtime-options NodeAutoFstrimEnabled=0
```

#### Disable on a cluster

To turn off auto fstrim for a cluster, run the following command:

```text
pxctl cluster options update --auto-fstrim off
```

## Manual filesystem trim operations

You can also perform filesystem trim operations manually using `pxctl`.

{{<info>}}
**NOTE:**

* To manually run filesystem trim operations, you need to disable auto fstrim on the node
* Filesystem trim operations can sometimes take a very long time to complete, so the service runs as a background operation
* You can only perform filesystem trim operations on a mounted volume
* If you unmount a volume while filesystem trim operations are running on it, those filesystem trim operations will stop
* You can only run 1 instance of filesystem trim at-a-time on a volume
* You can only run 1 instance of filesystem trim on a system. This limitation reduces the impact on IO performance for user workloads running on that node
* You must start filesystem trim operations from the node on which the volume's storage is mounted. For sharedv4 volumes, filesystem trim operation should be run on the nfs server node where the pxd volume is attached, mounted, and exported
{{</info>}}

### Perform a filesystem trim operation

1. Open a shell session with the Portworx node on which the volume you intend to run the filesystem trim operation on is mounted.

2. Enter the `pxctl volume trim start` command with the `--path` flag and your mount path and volume name to start the filesystem trim operation on a volume:

    ```text
    pxctl volume trim start --path <mount_path> <volume_name>
    ```

3. Monitor the filesystem trim operation running on a volume by entering the `pxctl volume trim status` command with the `--path` flag and your mount path and volume name:

    ```text
    pxctl volume trim status --path <mount_path> <volume_name>
    ```

### Stop a filesystem trim operation

Stop a running filesystem trim operation by entering the `pxctl volume trim status` command with the `--path` flag and your mount path and volume name:

```text
pxctl volume trim stop --path <mount_path> <volume_name>
```

### pxctl volume trim reference

#### pxctl volume trim start

```text
pxctl volume trim start --path <mount_path> <volume_name>
```

| Description | Arguments | Flags |
| --- | --- | --- |
Start a filesystem trim operation on the block device and volume you specify | `<volume_name>` | The name of the volume on which you want to perform a filesystem trim operation |  `--path`  | Use this flag to provide the mount path where the volume/device is mounted `<mount_path>` |

##### Example

Start a filesystem trim operation on an example volume:

```text
pxctl volume trim start --path /mnt/pxd/mount/path exampleVolume
```

#### pxctl volume trim status

```text
pxctl volume trim status --path <mount_path> <volume_name>
```

| Description | Arguments | Flags |
| --- | --- | --- |
| Display the status of a currently running filesystem trim operation on the block device and volume you specify | `<volume_name>` | The name of the volume you want to see the currently running filesystem trim operation status for | `--path`  | Use this flag and specify the path reference to the mount point or mount directory where the volume is mounted. |

##### Example

Show the status for a running filesystem trim operation on an example volume:

```text
pxctl volume trim status --path /mnt/pxd/mount/path exampleVolume
```

#### pxctl volume trim stop

```text
pxctl volume trim stop --path <mount_path> <volume_name>
```

| Description | Arguments | Flags |
| --- | --- | --- |
| Stop a currently running filesystem trim operation on the block device and volume you specify | `<volume_name>` | The name of the volume for which you want to stop a filesystem trim operation | `--path`  | Use this flag and specify the path reference to the mount point or mount directory where the volume is mounted, for example: `/var/lib/osd/examplevolume`. |

##### Example

Stop the running filesystem trim operation on an example volume:

```text
pxctl volume trim stop --path /mnt/pxd/mount/path exampleVolume
```
