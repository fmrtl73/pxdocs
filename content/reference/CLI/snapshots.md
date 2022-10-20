---
title: Snapshots operations using pxctl
linkTitle: Snapshots
keywords: pxctl, command-line tool, cli, reference, snapshot, create snapshot, list snapshots, delete snapshot, schedule policies, snapshot schedule
description: Learn how to manage snapshots using pxctl
weight: 400
---

{{< content "shared/reference-CLI-intro-snapshots.md" >}}

## Creating snapshots

{{< content "shared/reference-CLI-creating-snapshots.md" >}}

## Listing Snapshots

To list existing snapshots, you can use `pxctl volume list`. Let's have a look at the available flags:

```text
pxctl volume list --help
```

```output
List volumes in the cluster

Usage:
  pxctl volume list [flags]

Aliases:
  list, l

Flags:
  -a, --all                   show all volumes, including snapshots
  -g, --group string          show all volumes for given group
  -h, --help                  help for list
  -l, --label string          list of comma-separated name=value pairs
      --name string           volume name used during creation if any
      --node string           show all volumes whose replica is present on the given node
  -p, --parent string         show all snapshots created for given volume
      --pool-uid string       show all volumes whose replica is present on the given pool
      --sched-policy string   filter volumes by sched policy
  -s, --snapshot              show all snapshots (read-only volumes)
      --snapshot-schedule     show all schedule created snapshots
  -t, --time                  show all volumes sorted by creation time
  -v, --volumes               show only volumes

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx
```

### User created snapshots

To list your _user created snapshots_, use one of the following commands:

```text
pxctl volume list --all
```

```output
ID          NAME                                    SIZE    HA  SHARED  ENCRYPTED   COMPRESSED  IO_PRIORITY SCALE   STATUS
234835613696329810  mysnap                                  1 GiB   1   no  no      no      LOW     1   up - detached
1125771388930868153 myvol                                   1 GiB   1   no  no      no      LOW     1   up - detached
```

{{<info>}}
The above command shows **all volumes, including snapshots**.
{{</info>}}

\(or\)

```text
pxctl volume list --snapshot
```

```output
ID          NAME                                    SIZE    HA  SHARED  ENCRYPTED   COMPRESSED  IO_PRIORITY SCALE   STATUS
234835613696329810  mysnap                                  1 GiB   1   no  no      no      LOW     1   up - detached
```

{{<info>}}
The above command shows **only snapshots**.
{{</info>}}

### Scheduled snapshots

To list all your scheduled snapshots, run this command:

```text
pxctl volume list --snapshot-schedule
```

```output
ID          NAME                                    SIZE    HA  SHARED  ENCRYPTED   COMPRESSED  IO_PRIORITYSCALE    STATUS
423119103642927058  myvol_periodic_2018_Feb_26_21_12                    1 GiB   1   no  no      no      LOW     1up - detached
```

### Filtering the results

You can filter the results with the `–parent` and `–label` options. For instance, `–parent myvol` will show only snapshots whose parent is `myvol` (i.e. `mynsap`):

```text
pxctl volume list --parent myvol --snapshot
```

```output
ID          NAME    SIZE    HA  SHARED  ENCRYPTED   COMPRESSED  IO_PRIORITY SCALE   STATUS
234835613696329810  mysnap  1 GiB   1   no  no      no      LOW     1   up - detached
```

Giving labels restricts the list to snapshots that have all of the specified labels. For instance, –`label fabric=wool` would again show `mysnap` but `–label fabric=cotton` won't.

```text
pxctl volume list --parent myvol --snapshot --label fabric=wool
```

```output
ID          NAME    SIZE    HA  SHARED  ENCRYPTED   COMPRESSED  IO_PRIORITY SCALE   STATUS
234835613696329810  mysnap  1 GiB   1   no  no      no      LOW     1   up - detached
```

## Deleting snapshots

To delete a snapshot, run `pxctl volume delete` with the `name` or the `id` of the snapshot you want to delete as an argument:

```text
pxctl volume delete mysnap
```

```output
Delete volume 'mysnap', proceed ? (Y/N): y
Volume mysnap successfully deleted.
```

{{<info>}}
Only detached snapshots can be deleted.
{{</info>}}

## Restoring snapshots

{{< content "shared/reference-CLI-restore-volume-from-snapshot.md" >}}

## Snapshot schedule policies

You can use the `pxctl sched-policy` command to create and manage your snapshot schedule policies.

{{< content "shared/reference-CLI-sched-policy.md" >}}

For more information on how to create, list, update, and delete your snapshot schedule policies with `pxctl`, see [this page](/reference/cli/sched-policy).

## Snapshot schedules

If you create a volume and a snapshot schedule at the same time, you can use and combine as needed the following four scheduling options:

- –-periodic,
- –-daily,
- –-weekly and
- –-monthly.

Scheduled snapshots have names of the form `<Parent-Name>_<freq>_<creation_time>`, where `<freq>` denotes the schedule frequency (i.e., periodic, daily, weekly, monthly):

```
myvol_periodic_2018_Feb_26_21_12
myvol_daily_2018_Feb_26_12_00
```

As an example, to create a new volume named `myvol` and to schedule:

1. a periodic snapshot for every 60 min and a
2. daily snapshot at 8:00 am and a
3. weekly snapshot on Friday at 23:30 pm and
4. monthly snapshot on the 1st of the month at 6:00 am.

you would run this command:

```text
pxctl volume create --periodic 60 --daily @08:00 --weekly Friday@23:30 --monthly 1@06:00 myvol
```

Here's another example. In order to create a volume named `myvol` and to schedule:

1. 10 periodic snapshots that trigger every 120 min and
2. 3 daily snapshots that trigger at 8:00 am

you would run the following:

```text
pxctl volume create --periodic 120,10 --daily @08:00,3 myvol
```

{{<info>}}
Once the count is reached, the oldest existing one will be deleted if necessary.
{{</info>}}


### Changing a snapshot schedule

To change the snapshot schedule for a given volume, use the `pxctl volume snap-interval-update` command.

First, let's see the available flags:

```text
pxctl volume snap-interval-update --help
```

```output
Update volume configuration

Usage:
  pxctl volume snap-interval-update [flags]

Aliases:
  snap-interval-update, snap

Examples:
pxctl volume snap-interval-update [flags] volName

Flags:
  -d, --daily strings     daily snapshot at specified hh:mm,k (keeps 7 by default)
  -h, --help              help for snap-interval-update
  -m, --monthly strings   monthly snapshot at specified day@hh:mm,k (keeps 12 by default)
  -p, --periodic string   periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
      --policy string     policy names separated by comma
  -w, --weekly strings    weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx
```

In the below example, the old snapshot schedule is replaced with a daily snapshot triggered at 15:00 pm (5 snapshots are kept):

```text
pxctl volume snap-interval-update --daily @15:00,5 myvol
```

### Disabling scheduled snapshots

To disable scheduled snapshot for a given volume, use `--periodic 0` on `snap-interval-update`:

```text
pxctl volume snap-interval-update --periodic 0 myvol
```

### View the snapshot schedule for a volume

To view the snapshot schedule for a volume, use the `pxctl volume inspect` command as follows:

```text
pxctl volume inspect myvol
```

```output
Volume	:  1125771388930868153
	Name            	 :  myvol
	Size            	 :  1.0 GiB
	Format          	 :  ext4
	HA              	 :  1
	IO Priority     	 :  LOW
	Creation time   	 :  Feb 26 18:06:31 UTC 2018
	Snapshot        	 :  daily @15:00,keep last 5
	Shared          	 :  no
	Status          	 :  up
	State           	 :  Attached: minion1
	Device Path     	 :  /dev/pxd/pxd1125771388930868153
	Reads           	 :  54
	Reads MS        	 :  152
	Bytes Read      	 :  1105920
	Writes          	 :  53
	Writes MS       	 :  841
	Bytes Written   	 :  16891904
	IOs in progress 	 :  0
	Bytes used      	 :  48 MiB
	Replica sets on nodes:
		Set  0
			Node 		 :  70.0.34.84 (Pool 0)
	Replication Status	 :  Up

```

## Related topics

For information about creating snapshots of your Portworx volumes, refer to the [Create and use snapshots](/operations/operate-kubernetes/storage-operations/create-snapshots/) page.
