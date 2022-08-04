---
title: Install Portworx on Azure Red Hat OpenShift (ARO)
weight: 300
keywords: Install, on-premises, OpenShift, kubernetes, k8s, ARO, Azure, OpenShift
description: Install Portworx on Azure Red Hat OpenShift
linkTitle: Install on Azure Red Hat OpenShift
---

## Prerequisites

* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
* Logged in to your Azure account through the CLI (https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli)

## Procedure

* Follow the [cluster creation tutorial](https://docs.microsoft.com/en-us/azure/openshift/tutorial-create-cluster ) on the Microsoft Azure documentation, including the *Get a Red Hat pull secret (optional)* section.
* Follow the [cluster connection tutorial](https://docs.microsoft.com/en-us/azure/openshift/tutorial-connect-cluster) on the Microsoft Azure documentation.


### Find the ARO Service Principal

When deploying Portworx on Azure Redhat Openshift (ARO), the virtual machines are created in a resource group with a `Deny Assignment` role that prevents any service principal from accessing virtual machines except the service principal created for the resource group. In this task, you identify the service principal for the resource group that has access, and configure it to pass on the credentials (Azure Client ID, Azure Client Secret and Tennant ID) via the Portworx cluster spec. Portworx will fetch the `px-azure` secret object file to authenticate.


1. From the Azure Web UI, select *Virtual Machines* in the upper right corner.
2. From the *Virtual machines* page, select the *Resource Group* associated with your cluster.
3. From the left panel on the *Resource group* page, select *Access control (IAM)*.
4. On the *Access control (IAM)* subpage of your resource group, select *Deny assignments* from the toolbar in the center of the page, then select the link under the *Name* column (this will likely be an autogenerated string of letters and numbers).
5. This page shows that all principals are denied access, except for your resource group. Select your resource group's name.
6. From the application page, copy and save the following values: 
    * Name
    * Application ID
    * Object ID
    
    You will use these to create the `px-azure` secret.

7. From the home page, open the *Azure Active Directory* page (select *All services* to see the option). Select *App registrations* on the left pane, followed by *All applications*. In the search bar in the center of the page, paste the application name you saved in the previous step and hit the enter key. Select the application link that shows in the results to open the next page.
8. From your application's page, select *Certificates & secrets* from the left pane. 
9. From the *Certificates & secrets* page, select *+ New client secret* to create a new secret. 
10. Use the Application ID and Object ID values from step 6 to create a new secret, and save this secret for future use.

### Create the px-azure secret with Service Principal credentials

Create a secret called `px-azure` to give Portworx access to Azure APIs by updating the following fields with the associated fields from the service principal you created in the step above:

```text
kubectl create secret generic -n kube-system px-azure --from-literal=AZURE_TENANT_ID=<tenant> --from-literal=AZURE_CLIENT_ID=<appId> --from-literal=AZURE_CLIENT_SECRET=<password>
```
```output
secret/px-azure created
```

## Deploy Portworx

1. Go to [PX-Central](https://central.portworx.com/) to generate an installation spec. 

2. Click **Continue** with Portworx Enterprise option.
  ![PX option screen](/img/aro/image1.png)

3. Choose an appropriate license for your requirement and click **Continue**.
  ![PX license screen](/img/aro/image2.png)

4. Ensure that the **Use the Portworx Operator** option is selected and click **Next**.
  ![Basic screen](/img/aro/image3.png)

5. Select **Cloud** as your environment, **AZURE** as cloud platform, and click **Next**.
  ![Your Environment](/img/aro/image4.png)

6. Choose your network and click **Next**.
  ![Network](/img/aro/image5.png)

7. Select **Azure Kubernetes Service (AKS)** option on the **Customize** page, and click **Finish**.
  ![Other environment](/img/aro/image6.png)

8. Follow the instructions to install Portworx Operator.
  ![PX-Operator](/img/aro/Operator-install.png)

8. Download the Portworx spec as a YAML file, and append the `osft=true` and `portworx.io/is-openshift: "true"` values. Following is a sample spec for your reference.

    ```text   
    # SOURCE: https://install.portworx.com/?operator=true&mc=false&kbver=&b=true&kd=type%3DPremium_LRS%2Csize%3D150&s=%22type%3DPremium_LRS%2Csize%3D150%22&c=px-cluster-068edac2-6b76-4a58-9227-bbeccb6c0928&aks=true&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true
    kind: StorageCluster
    apiVersion: core.libopenstorage.org/v1
    metadata:
      name: px-cluster-068edac2-6b76-4a58-9227-bbeccb6c0928
      namespace: kube-system
      annotations:
        portworx.io/install-source: "https://install.portworx.com/?operator=true&mc=false&kbver=&b=true&kd=type%3DPremium_LRS%2Csize%3D150&s=%22type%3DPremium_LRS%2Csize%3D150%22&c=px-cluster-068edac2-6b76-4a58-9227-bbeccb6c0928&aks=true&osft=true&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true"
        portworx.io/is-aks: "true"
        portworx.io/is-openshift: "true"
    spec:
      image: portworx/oci-monitor:2.10.3
      imagePullPolicy: Always
      kvdb:
        internal: true
      cloudStorage:
        deviceSpecs:
        - type=Premium_LRS,size=150
        kvdbDeviceSpec: type=Premium_LRS,size=150
      secretsProvider: k8s
      stork:
        enabled: true
        args:
          webhook-controller: "true"
      autopilot:
        enabled: true
      monitoring:
        prometheus:
          enabled: true
          exportMetrics: true
      featureGates:
        CSI: "true"
      env:
      - name: AZURE_CLIENT_SECRET
        valueFrom:
          secretKeyRef:
            name: px-azure
            key: AZURE_CLIENT_SECRET
      - name: AZURE_CLIENT_ID
        valueFrom:
          secretKeyRef:
            name: px-azure
            key: AZURE_CLIENT_ID
      - name: AZURE_TENANT_ID
        valueFrom:
          secretKeyRef:
            name: px-azure
            key: AZURE_TENANT_ID
    ```

9. Use the `kubectl -apply <portworx_enterprise.yaml>` command to deploy Portworx.
  
{{< content "shared/portworx-install-with-kubernetes-post-install.md" >}}






￼

