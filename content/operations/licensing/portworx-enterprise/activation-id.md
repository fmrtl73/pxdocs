---
title: Activate your license using an Activation ID
linkTitle: Enable using Activation ID 
description: Learn about the Portworx Enterprise 
keywords: enterprise license, license sharing, transfer license, air-gapped license, Saas Key, Licensing, Portworx Enterprise, upgrade Portworx, activate license
weight: 100
aliases:
    - /reference/licensing/portworx-enterprise/activation-id/
---
The easiest way to install the {{< pxEnterprise >}} license, is via Activation ID (contact Portworx for getting Activation ID). 

 Run the following command on your Portworx node:

```text
pxctl license activate <activation-id>
```

The license activation process requires an active internet connection from the Portworx nodes to the license server.
The activation process automatically registers the cluster UUID, and generates and installs the license on the cluster.
Upon activating the license on one Portworx node, all the remaining Portworx nodes automatically update to the new license.

{{<info>}}
**NOTE:** You can also run the `pxctl license activate` command inside a pod as follows:

 ```text
 PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
 kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl license activate <activation-id>
 ```
{{</info>}}
