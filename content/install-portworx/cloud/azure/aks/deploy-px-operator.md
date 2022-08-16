---
title: Install Portworx on AKS
linkTitle: Install Portworx
keywords: on cloud, AKS, Azure Kubernetes Service, Microsoft, Kubernetes, k8s
description: Learn about applying the spec with Portwork on Azure Kubernetes Service.
weight: 300
aliases:
    - /install-portworx/cloud/azure/aks/deploy-px-daemonset/
---

## Install

{{< content "shared/operator-install.md" >}}

{{< content "shared/portworx-install-with-kubernetes-shared-generate-the-spec-footer-operator.md" >}}

{{<info>}}
**NOTE:** To deploy Portworx to an Azure Sovereign cloud, you must go to the **Customize** page and set the value of the `AZURE_ENVIRONMENT` variable. The following example screenshot shows how you can deploy Portworx to the Azure US Government cloud:

![Screenshot showing the AZURE_ENVIRONMENT variable](/img/azure-sovereign-example.png)

{{</info>}}

### (Optional) Enable Azure cloud drive encryption using your own key

 You can encrypt your Azure cloud drives that are managed by Portwox by using your own key stored in Azure Key Vault.
 
**Prerequisites**

Azure KeyVault instance created in the same region as the AKS cluster. 

**Procedure**

1. Create a Disk Encryption Set ID by using the instructions on [this page](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-enable-customer-managed-keys-portal) in the Microsoft documentation.

2. Append the `diskEncryptionSetID` value from step 1 to the spec and deploy Portworx using the updated spec:

    ```text
    cloudStorage:
        deviceSpecs:
        - type=Premium_LRS,size=50,diskEncryptionSetID=/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Compute/diskEncryptionSets/<disk-encryption-set-name>
    secretsProvider: azure-kv
    ```
    

{{< content "shared/operator-apply-the-spec.md" >}}

{{< content "shared/operator-monitor.md" >}}

{{< content "shared/portworx-install-with-kubernetes-post-install.md" >}}
