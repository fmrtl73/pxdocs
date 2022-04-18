---
title: CloudUserRequirements
description:
keywords:
hidden: true
---

### Create a custom role for Portworx

```text
az role definition create --role-definition '{
        "Name": "portworx-cloud-drive",
        "Description": "",
        "AssignableScopes": [
            "/subscriptions/72c299a4-xxxx-xxxx-xxxx-6855109979d9"
        ],
        "Permissions": [
            {
                "Actions": [
                    "Microsoft.ContainerService/managedClusters/agentPools/read",
                    "Microsoft.Compute/disks/delete",
                    "Microsoft.Compute/disks/write",
                    "Microsoft.Compute/disks/read",
                    "Microsoft.Compute/virtualMachines/write",
                    "Microsoft.Compute/virtualMachines/read",
                    "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/write",
                    "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read"
                ],
                "NotActions": [],
                "DataActions": [],
                "NotDataActions": []
            }
        ]
}'
```

### Create a Service Principal and secret in Azure AD

1. Find the AKS cluster Infrastructure Resource Group:

    ```text
    az aks show -n <aks-cluster-name> -g <aks-resource-group> | jq -r '.nodeResourceGroup'
    ```

1. Create a service principal for Portworx with the custom role:

    ```text
    az ad sp create-for-rbac --role=portworx-cloud-drive --scopes="/subscriptions/72c299a4-xxxx-xxxx-xxxx-6855109979d9/resourceGroups/<aks-infrastructure-resource-group>"
    ```
    ```output
    {
      "appId": "1311e5f6-xxxx-xxxx-xxxx-ede45a6b2bde",
      "displayName": "azure-cli-2020-10-10-10-10-10",
      "name": "http://azure-cli-2020-10-10-10-10-10",
      "password": "ac49a307-xxxx-xxxx-xxxx-fa551e221170",
      "tenant": "ca9700ce-xxxx-xxxx-xxxx-09c48f71d0ce"
    }
    ```

2. Create a secret called `px-azure` to give Portworx access to Azure APIs by updating the following fields with the associated fields from the service principal you created in the step above:


    ```text
    kubectl create secret generic -n kube-system px-azure --from-literal=AZURE_TENANT_ID=<tenant> \
                                                          --from-literal=AZURE_CLIENT_ID=<appId> \
                                                          --from-literal=AZURE_CLIENT_SECRET=<password>
    ```
    ```output
    secret/px-azure created
    ```

Now that you've created the secret, you're ready to create the spec and deploy Portworx. The spec generator automatically incorporates the secret that you created, and Portworx will fetch the secret to authenticate. Proceed to the next section to install Portworx.