---
title: Restoring Volumes from Snapshots
hidden: true
keywords: portworx, pxctl, command-line tool, cli, reference, snapshot, restore volume
description: Learn how to restore volumes from snapshots
---

In order to restore a volume from snapshot use the `pxctl volume restore` command:

```text
/opt/pwx/bin/pxctl volume restore -h
```

```output
Restore volume from snapshot

Usage:
  pxctl volume restore [flags]

Aliases:
  restore, r

Examples:
pxctl volume restore [flags] volName

Flags:
  -h, --help              help for restore
  -s, --snapshot string   snapshot-name-or-ID

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

In the below example parent volume `myvol` is restored from its snapshot `mysnap`. Make sure volume is detached in order to restore from the snapshot.

```text
pxctl volume restore --snapshot mysnap myvol
```

```output
Successfully started restoring volume myvol from mysnap.
```

To restore a volume from the [trash can](/reference/cli/trashcan/), specify the --trashcan flag:

```text
pxctl volume restore --trashcan trashedvol myvol
```

```output
Successfully started restoring volume myvol from trashedvol.
```