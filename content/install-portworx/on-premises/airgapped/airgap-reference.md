---
title: Air-gapped install bootstrap script reference
linkTitle: Air-gapped install bootstrap script reference
weight: 300
logo: /logos/other.png
keywords: Install, on-premises, kubernetes, k8s, air gapped
description: Reference for the installation script used when installing or upgrading Portworx on an air-gapped cluster
noicon: true
aliases:
    - /portworx-install-with-kubernetes/on-premise/airgapped/airgap-reference/
---

```text
./px-ag-install.sh [image-commands] [options] '[arguments...]'
```

{{<info>}}
**NOTE:** The script name `px-ag-install.sh` reflects the default name specified in the installation instructions, but can be whatever you named the script file when you downloaded it.
{{</info>}}

### Image commands

|**Command**|**Description**|**Required?**|
|----|----|----|
| `pull` | Pulls the Portworx container images locally | |
| `push <registry[/repo]>` | Pushes the Portworx images into remote container registry server | |
| `load node1 [node2 [...]]` | Loads the images tarball to remote nodes  (note: ssh-access required) | |

### Options

|**Option**|**Description**|**Required?**|
|----|----|----|
| `--help` | Displays help output | |
| `-I`, `--include <image>` | Specify additional images to include | |
| `-E`, `--exclude <glob>` | Specify images to EXCLUDE  (e.g. -E '*csi*') | |
| `-n`, `--dry-run` | Show commands instead of running | |
| `-V`, `--version` | Print version of the script | |
| `-v` | Verbose output | |

### Load-specific options

|**Option**|**Description**|**Required?**|
|----|----|----|
| `-e`, `--rsh <command>`       | specify the remote shell to use  (default ssh) |
| `-L`, `--load-cmd <command>`  | specify the remote container-load command to use  (default auto) |
| `-t <prefix>`              | specify temporary tarball filename  (default px-agtmp.tar) |
| `--pks`                    | assume PKS environment; transfer images using 'bosh' command |

### Examples

* Pull images from default container registries, push them to custom registry server (default repositories)

    ```text
    px-ag-install.sh pull push your-registry.company.com:5000
    ```

* Pull images from default container registries, push them to custom registry server and portworx repository

    ```text
    px-ag-install.sh pull
    px-ag-install.sh push your-registry.company.com:5000/portworx
    ```

* Push images to password-protected remote registry, then import docker/podman configuration as kuberentes secret

    ```text
    docker login your-registry.company.com:5000
    px-ag-install.sh pull
    px-ag-install.sh push your-registry.company.com:5000/portworx
    px-ag-install.sh import-secrets
    ```

* Pull images, then load to given nodes using ssh

    ```text
    px-ag-install.sh pull
    px-ag-install.sh load node1 node2 node33 node444
    ```

* Pull images, then load to given nodes using ssh and root-account

    ```text
    px-ag-install.sh -e "ssh -l root" pull load node1 node2 node33 node444
    ```

* Load images to given nodes using ssh and password '5ecr3t'

    ```text
    px-ag-install.sh -e "sshpass -p 5ecr3t ssh" load node1 node2 node33 node444
    ```

* Pull ONLY busybox image, load it to given nodes

    ```text
    px-ag-install.sh -E '*' -I docker.io/busybox:latest pull load node1 node2 node33 node444
    ```
