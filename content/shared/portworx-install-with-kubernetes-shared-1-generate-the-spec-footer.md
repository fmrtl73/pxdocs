---
title: Shared
hidden: true
keywords: portworx, kubernetes
description: Learn how to install Portworx with Kubenetes
---
### Generate the specs

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

1. Navigate to <a href="https://central.portworx.com" target="tab">PX-Central</a> and log in, or create an account.

2. Click **Continue** with Portworx Enterprise option:

    ![Screenshot showing install and run](/img/pxcentral-install.png)

3. Choose an appropriate license for your requirement and click **Continue**:

    ![Screenshot showing Portworx license selector](/img/pxcentral-license.png)

{{<info>}}**NOTE**: If you're using a cloud provider, **do not add volumes of different types** when configuring storage devices for during spec generation. For example, do not add both GP2 and GP3 for AWS, standard and ssd for GCP, or Standard and Premium for Azure. This can cause performance issues and errors.{{</info>}}

Portworx can also be installed using its Helm chart by following instructions [here](/portworx-install-with-kubernetes/install-px-helm). The above method is recommended over helm as the wizard will guide you based on your environment.
