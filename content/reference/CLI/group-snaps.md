---
title: Group Snaps using pxctl
keywords: pxctl, command-line tool, cli, reference, group snapshots
description: Explore the CLI reference guide for taking group snapshots of container data volumes using Portworx. Try it today!
linkTitle: Group Snaps
weight: 500
---

This document explains how to take group snapshots of your container data with Portworx.

First, let's get an overview of the available flags before diving in:

```text
pxctl volume snapshot group --help
```

```output
Create group snapshots for given group id or labels

Usage:
  pxctl volume snapshot group [flags]

Aliases:
  group, g

Flags:
  -d, --delete-on-failure   delete created snaps if not all volumes succeeded in the group
  -g, --group string        group id
  -h, --help                help for group
  -l, --label string        list of comma-separated name=value pairs
  -v, --volume_ids string   list of comma-separated volume IDs

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

To take a group snapshot of the volumes labelled with `v1=x1`, use this command:

```text
pxctl volume snapshot group --label v1=x1
```

```output
Volume 549285969696152595 : Snapshot 1026872711217134654
Volume 952350606466932557 : Snapshot 218459942880193319
```

You can easily group volumes by IDs and take a group snapshot with the `--volume_ids` flag:

```text
pxctl volume snapshot group --volume_ids 83958335106174418,874802361339616936
```

```output
Volume 83958335106174418 : Snapshot 362408823552094597
Volume 874802361339616936 : Snapshot 895516478416742770
```

## Related topics

* For more information about creating group snapshots of Portworx volumes through Kubernetes, refer to the [Snapshot group of PVCs](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-group/) page.
