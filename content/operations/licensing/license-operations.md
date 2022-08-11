---
title: Licensing operations
keywords: Licensing, check your license, list license, upgrade Portworx,
description: Find out more about the Portworx licensing model.
weight: 100
aliases:
    - /reference/licensing/license-operations/
---
## Check your license

A brief license summary is provided with the `pxctl status` command:

```text
pxctl status
```

```output
Status: PX is operational
License: Trial license (expires in 30 days)
 [...]
```

Run the `pxctl license list` command to see more details about each licensed feature. For example,

```text
pxctl license list
```

```output
DESCRIPTION                  ENABLEMENT  ADDITIONAL INFO
Number of nodes maximum         1000
Number of volumes maximum       1024 [...]
Virtual machine hosts            yes
Product SKU                     Trial    expires in 30 days
```

{{<info>}}
**NOTE:** 7 days before your Portworx license is set to expire, Portworx sends an alert notifying your license is about to expire and how long you have to renew your license. The alert will keep triggering after the license has expired. For more details, see [this page](/operations/operate-other/monitoring/portworx-alerts#list-of-alerts).
{{</info>}}

## List available licenses

You can use `pxctl license list` to list installed licenses as follows:

```text
pxctl license list
```

```output
DESCRIPTION				ENABLEMENT	ADDITIONAL INFO
Number of nodes maximum			1000
Number of volumes maximum		100000
Volume capacity [TB] maximum		  40
Storage aggregation			 yes
Shared volumes				 yes
Volume sets				 yes
BYOK data encryption			 yes
Resize volumes on demand		 yes
Snapshot to object store [CloudSnap]	 yes
Cluster-level migration [Kubemotion]	 yes
Bare-metal hosts			 yes
Virtual machine hosts			 yes
Product SKU				Trial		expires in 6 days, 12:13

LICENSE EXPIRES: 2019-04-07 23:59:59 +0000 UTC
For information on purchase, upgrades and support, see
https://docs.portworx.com/knowledgebase/support.html
```

As you can see, the command gives details on the features allowed under the current licenses, and it lists the SKU.

## Related topics

* [Portworx Essentials license](/operations/licensing/portworx-essential)
* [Portworx Enterprise license](/operations/licensing/portworx-enterprise)
* [License operations using pxctl](/reference/cli/license)

