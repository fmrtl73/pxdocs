---
title: OS Configurations for Shared Mounts
keywords: portworx, px-developer, shared mounts
description: Here is the step-by-step tutorial to share mounts using Portworx. We can help with CoreOS, RedHat/Centos and Ubuntu. See for yourself today!
weight: 2
linkTitle: Shared Mount Propagation
---

Portworx requires Docker to allow shared mounts.

### Checking whether shared mounts are enabled
If the following command succeeds, shared mounts are enabled on your system and no action is needed.

```bash
$ docker run -it -v /mnt:/mnt:shared busybox sh -c /bin/date
```
If the command fails with below error, you need to enable shared mounts using this guide.

```bash
$ docker run -it -v /mnt:/mnt:shared busybox sh -c /bin/date
docker: Error response from daemon: linux mounts: Path /mnt is mounted on / but it is not a shared mount..
```

The following sections describe how to configure Docker for shared mounts on [CoreOS](#coreos-configuration-and-shared-mounts),
[RedHat/CentOS](#redhatcentos-configuration-and-shared-mounts),
and [Ubuntu](#ubuntu-configuration-and-shared-mounts). The configuration is required because the Portworx solution exports mount points.

### CoreOS Configuration and Shared Mounts

1. Verify that your Docker version is 1.10 or later:
```bash
$ docker -v
```

2. Copy the docker.service file for editing:
```bash
$ sudo cp /usr/lib64/systemd/system/docker.service /etc/systemd/system
```

3. Edit the docker service file for `systemd`:
```bash
$ sudo vi /etc/systemd/system/docker.service
```

4. Remove the `MountFlags` line.

5. Reload the daemon:
```bash
$ sudo systemctl daemon-reload
```

6. Restart Docker:
```bash
$ sudo systemctl restart docker
```

### RedHat/CentOS Configuration and Shared Mounts

1. Follow the Docker installation guide for [Red Hat Enterprise Linux](https://www.docker.com/docker-red-hat-enterprise-linux-rhel)/[Centos](https://docs.docker.com/engine/installation/linux/centos/) and then start the Docker service.

2. Verify that your Docker version is 1.10 or later:
```bash
$ docker -v
```

3. Edit the service file for `systemd`:
```bash
$ sudo vi /lib/systemd/system/docker.service
```
4. Remove the `MountFlags` line.

5. Reload the daemon:
```bash
$ sudo systemctl daemon-reload
```
6. Restart Docker:
```bash
$ sudo systemctl restart docker
```

### Ubuntu Configuration and Shared Mounts

1. SSH into your first server.
2. Follow the Docker installation guide for [Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/):

3. Verify that your Docker version is 1.10 or later. In your SSH window, run:
```bash
$ docker -v
```
4. In your SSH window, run:
```bash
$ sudo mount --make-shared /
```

5. If you are using `systemd`, remove the `MountFlags=slave` line in your docker.service file.

    The Ubuntu in this example is not using `systemd`.

### Amazon Linux Configuration and Shared Mounts

NOTE that Amazon Linux EC2 images do not have the [`systemd(1)`](http://man7.org/linux/man-pages/man1/systemd.1.html) service, so the installation instructions are somewhat different:

1. Using [Amazon EC2](https://aws.amazon.com/ec2/), start your Amazon Linux instance (`Amazon Linux AMI 2016.09.1, ami-f173cc91`), and SSH into it.
2. Install docker package:
```bash
$ sudo yum install docker
```

3. Verify that your Docker version is 1.10 or later. In your SSH window, run:
```bash
$ docker -v
```

4. In your SSH window, run:
```bash
$ sudo mount --make-shared /
```

5. Add the `--propagation shared` flags to the docker startup script:
```bash
$ sudo sed -i.bak -e \
's:^\(\ \+\)"$unshare" -m -- nohup:\1"$unshare" -m --propagation shared -- nohup:' \
	/etc/init.d/docker
```

6. Restart docker service:
```bash
$ sudo service docker restart
```

### Verify that shared mounts work

Head back to [Checking whether shared mounts are enabled](#checking-whether-shared-mounts-are-enabled) to check that shared mounts are now working.
