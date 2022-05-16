---
title: Deploy Portworx using Azure managed identity on new AKS cluster
keywords: Install, deploy Portworx using Azure managed identity, AKS cluster, Azure Kubernetes Service, Kubernetes, k8s
description: Learn about deploying Portworxon using Azure managed identity on new AKS cluster.
weight: 100
linkTitle: Azure managed identity on AKS
---

Perform the following steps to enable Azure managed identity on new AKS cluster:

1. Login to the Azure and set the subscription:

    ```text
    az login
    az account set –subscription <Your-Azure-Subscription-UUID>
    ```

2. Check locations to create AKS cluster:

    ```text
    az account list-locations
    ```

3. Create an Azure Resource Group:

    ```text
    az group create –name <resource-group-name> –location <location>
    ```

4. Create an AKS cluster with managed identities:

    ```text
    az aks create -g myResourceGroup -n myManagedCluster --enable-managed-identity
    ```

5. Identify object and client IDs:

    ```text
    az aks show -g myResourceGroup -n myManagedCluster --query identityProfile
    ```

    Example:

    ```text
    az aks show -g cass-rg -n msi-test --query identityProfile
    {
      "kubeletidentity": {
        "clientId": "68c2bc67-f3a5-459d-9b57-14597efcbc70",
        "objectId": "c099f8ac-ba91-4c13-9456-3e5614296a35",
        "resourceId": "/subscriptions/72c299a4-a431-4b8e-80ef-6855109979d9/resourcegroups/MC_cass-rg_msi-test_eastus/providers/Microsoft.ManagedIdentity/userAssignedIdentities/msi-test-agentpool"
      }
    }
    ```

6. Assign contributor role to managed identity:

    ```text
     az aks show -g cass-rg -n msi-test --query nodeResourceGroup
     az role assignment create --assignee-object-id ObjectId --role "Contributor" --resource-group nodeResourceGroup
    ```

    Example:

    ```text
    az aks show -g myResourceGroup -n myManagedCluster --query nodeResourceGroup
    "MC_cass-rg_msi-test_eastus"
    az role assignment create --assignee-object-id "c099f8ac-ba91-4c13-9456-3e5614296a35" --role "Contributor" --resource-group "MC_cass-rg_msi-test_eastus"
    {
      "canDelegate": null,
      "condition": null,
      "conditionVersion": null,
      "description": null,
      "id": "/subscriptions/72c299a4-a431-4b8e-80ef-6855109979d9/resourceGroups/MC_cass-rg_msi-test_eastus/providers/Microsoft.Authorization/roleAssignments/d0060dc6-4e9f-452c-8e43-1a661ecf0111",
      "name": "d0060dc6-4e9f-452c-8e43-1a661ecf0111",
      "principalId": "c099f8ac-ba91-4c13-9456-3e5614296a35",
      "principalType": "ServicePrincipal",
      "resourceGroup": "MC_cass-rg_msi-test_eastus",
      "roleDefinitionId": "/subscriptions/72c299a4-a431-4b8e-80ef-6855109979d9/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
      "scope": "/subscriptions/72c299a4-a431-4b8e-80ef-6855109979d9/resourceGroups/MC_cass-rg_msi-test_eastus",
      "type": "Microsoft.Authorization/roleAssignments"
    } 
    ```

7. Create Kubernetes secret based on Client ID shown above:

    ```text
    kubectl create secret generic -n kube-system px-azure --from-literal=AZURE_CLIENT_ID=clientId
    ```

    Example:

    ```text
    kubectl create secret generic -n kube-system px-azure --from-literal=AZURE_CLIENT_ID="68c2bc67-f3a5-459d-9b57-14597efcbc70”
    ```

8. Follow the steps to generate the Operator and StorageCluster spec in [Install Portworx on AKS using the Operator](/portworx-install-with-kubernetes/cloud/azure/aks/deploy-px-operator/). Save the spec for the next step.

9. Modify the StorageCluster spec that is automatically generated. In the `env` section, remove the `AZURE_CLIENT_SECRET` and `AZURE_TENANT_ID` sections. The finished section should match the following: 

    ```text
    env:
    name: AZURE_CLIENT_ID
      valueFrom:
        secretKeyRef:
          name: px-azure
          key: AZURE_CLIENT_ID
    ```
