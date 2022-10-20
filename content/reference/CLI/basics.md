---
title: Basics operations using pxctl
description: General reference for CLI, Volumes and other resources.
keywords: pxctl, command-line tool, cli, basics, reference
weight: 200
linkTitle: Basics
---

In this document, we are going to explore the basic operations available through the Portworx command-line tool- `pxctl`.

By default, the CLI displays the information in human readable form. For example, to learn more about the available commands, type `pxctl --help`:

```text
pxctl --help
```

```output
px cli

Usage:
  pxctl [command]

Available Commands:
  alerts         px alerts
  auth           pxctl auth
  clouddrive     Manage cloud drives
  cloudmigrate   Migrate volumes across clusters
  cloudsnap      Backup and restore snapshots to/from cloud
  cluster        Manage the cluster
  context        pxctl context
  credentials    Manage credentials for cloud providers
  eula           Show license agreement
  help           Help about any command
  license        Manage licenses
  role           pxctl role
  sched-policy   Manage schedule policies
  secrets        Manage Secrets. Supported secret stores AWS KMS | Vault | DCOS Secrets | IBM Key Protect | Kubernetes Secrets | Google Cloud KMS
  service        Service mode utilities
  status         Show status summary
  storage-policy Manage storage policies for creating volumes
  upgrade        Upgrade PX
  volume         Manage volumes

Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -h, --help                 help for pxctl
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx
  -v, --version              print version and exit

Use "pxctl [command] --help" for more information about a command.
```

{{<info>}}
As seen above, `pxctl` provides the capability to perform fine-grained control of the Portworx resources cluster-wide. Also, it lets the user manage volumes, snapshots, cluster resources, hosts in the cluster and software upgrade in the cluster.
{{</info>}}

In addition, every command takes in a `--json` flag which converts the output to a machine-parsable `JSON` format. You can do something like the following to save the output in `JSON` format:

```text
pxctl status --json > status.json
```

In most production deployments, you will provision volumes directly using _Docker_ or your scheduler (such as a _Kubernetes_ pod spec). However, `pxctl` also lets you directly provision and manage storage. In addition, `pxctl` has a rich set of cluster-wide management features which are explained in this document.

All operations available through `pxctl` are reflected back into the containers that use Portworx storage. In addition to what is exposed in Docker volumes, `pxctl`:

*   Gives access to Portworx storage-specific features, such as cloning a running container’s storage.
*   Shows the connection between containers and their storage volumes.
*   Lets you control the Portworx storage cluster, such as adding nodes to the cluster. (The Portworx tools refer to servers managed by Portworx storage as _nodes_.)

The scope of the `pxctl` command is global to the cluster. Running `pxctl` on any node within the cluster, therefore, shows the same global details. But `pxctl` also identifies details specific to that node.

The current release of `pxctl` is located in the `/opt/pwx/bin/` directory of every **worker node** and requires that you run it as a privileged user.

Let's look at some simple commands.

## Version

Here's how to find out the current version:

```text
pxctl --version
```

```output
pxctl version 2.7.0.0-886586d
```

## Status

This section has been moved to the [status page](/reference/cli/status).

## Upgrade related operations

`pxctl` provides access to several upgrade related operations. You can get details on how to use it and of the available flags by running:

```text
pxctl upgrade --help
```

```output
Usage:
  pxctl upgrade [flags]

Flags:
  -l, --tag string   Specify a PX Docker image tag
  -h, --help         help for upgrade

Global Flags:
      --ca string        path to root certificate for ssl usage
      --cert string      path to client certificate for ssl usage
      --color            output with color coding
      --config string    config file (default is $HOME/.pxctl.yaml)
      --context string   context name that overrides the current auth context
  -j, --json             output in json
      --key string       path to client key for ssl usage
      --raw              raw CLI output for instrumentation
      --ssl              ssl enabled for portworx
```

### Running pxctl upgrade

`pxctl upgrade` upgrades the Portworx version on a node. Let's suppose you want to upgrade Portworx to version _1.1.16_. If so, you would then type the following command:

```text
pxctl upgrade --tag 1.1.6 my-px-enterprise
```

```output
Upgrading my-px-enterprise to version: portworx/px-enterprise:1.1.6
Downloading PX portworx/px-enterprise:1.1.6 layers...
<Output truncated>
```

It is recommended to upgrade the nodes in a **staggered manner**. This way, the quorum and the continuity of IOs will be maintained.

### Related topics

* For information about upgrading Portworx through Kubernetes, refer to the [Upgrade on Kubernetes](/operations/operate-kubernetes/upgrade/) page.
* If you’re using the Portworx Operator, refer to the [Upgrade Portworx using the Operator](/install-portworx/openshift/operator/openshift-upgrade/) page.

## Login/Authentication

You must make Portworx login to the secrets endpoint when using encrypted volumes and ACLs.

`pxctl secrets` can be used to configure authentication credentials and endpoints.
Currently, Vault, Amazon KMS, and KVDB are supported.


### Vault example

Here's an example of configuring Portworx with Vault:

```text
pxctl secrets vault login --vault-address http://myvault.myorg.com --vault-token myvaulttoken
```

```output
Successfully authenticated with Vault.
```

{{<info>}}
To install and configure Vault, peruse [this link](https://www.vaultproject.io/docs/install)
{{</info>}}

### AWS KMS example

To configure Portworx with Amazon KMS, type the following command:

```text
pxctl secrets aws login
```

Then, you will be asked a few questions:

```
Enter AWS_ACCESS_KEY_ID [Hit Enter to ignore]: ***
Enter AWS_SECRET_ACCESS_KEY [Hit Enter to ignore]: ***
Enter AWS_SECRET_TOKEN_KEY [Hit Enter to ignore]: ***
Enter AWS_CMK [Hit Enter to ignore]: mykey
Enter AWS_REGION [Hit Enter to ignore]: us-east-1b
```

Finally, a success message will be displayed:

```
Successfully authenticated with AWS.
```

### EULA

You can get a link to our EULA by running:

```text
pxctl eula
```

```output
https://portworx.com/end-user-license-agreement
```
