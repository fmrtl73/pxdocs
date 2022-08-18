---
title: Dynamic volume creation through DCOS
description: Read the overview on how volumes can be created dynamically through DCOS. Gain a better understanding of dynamic volumes here!
keywords: inline volume spec, dynamic volumes, DCOS, DC/OS, Mesosphere, Marathon
hidden: true
linkTitle: Dynamically Creating Volumes
aliases:
    - /install-with-other/dcos/operate-and-maintain/inline/
---
## Inline volume spec
Portworx supports passing the volume spec inline along with the volume name. This is useful when creating a volume with DCOS through a marathon application template. Using the inline spec, volumes can be created dynamically and all Portworx properties, such as volume size, encryption keys etc can be passed in through marathon.

For example, a Portworx inline spec can be specified as the following:

```text
"parameters": [
	{
		"key": "volume-driver",
		"value": "pxd"
	},
	{
		"key": "volume",
		"value": "size=100G,repl=3,io_priority=high,name=mysql_vol:/var/lib/mysql"
	}],
```

The above command will create a volume called mysql_vol with an initial size of 100G, HA factor of 3 and a IO priority level of 3 for the mysql container.

Each spec key must be comma separated.  The following are supported key value pairs:

```text
IO priority      - io_priority=[1,2,3] or [high,medium,low]
Volume size      - size=[1..9][G|M|T]
HA factor        - repl=[1,2,3]
Block size       - bs=[4096...]
Shared volume    - sharedv4=true
File System      - fs=[xfs|ext4]
Encryption       - passphrase=secret
```

These inline specs can be passed in through the scheduler application template or framework.