---
title: Portworx on Equinix Metal with Google Anthos
keywords: Install, Equinix Metal, Terraform, Anthos, bare metal
description: Portworx can help you run stateful workloads with Docker. Find out how to deploy upon Packet.net servers!
weight: 600
linkTitle: Equinix Metal
noicon: true
aliases:
    - /install-with-other/other-clouds/packet/
    - /portworx-install-with-kubernetes/cloud/metal
---

The instructions in this document allow you to deploy a Kubernetes cluster with Portworx on Equinix Metal servers. In this deployment approach, you'll use  Google Anthos as the management infrastructure and Hashicorp Terraform to deploy everything. 

By using Terraform, you avoid the complexity of manually installing and configuring each of these components one-by-one. Instead, you can define certain aspects of how the cluster is deployed by creating a configuration file and let the Terraform do the rest. 

## Prerequisites

* A Google Cloud Platform (GCP) account with an active Google Anthos subscription (you'll need to know your GCP project ID)
* An Equinix metal account (you'll need your Metal auth token and Metal project ID)
* Terraform and the `gcloud` command-line tool installed on your client

## Deploy Portworx on Equinix Metal with Google Anthos

1. Create or select your desired GCP project using `gcloud init`:
   ```text
   gcloud init
   ```

2. Save your GCP user access credentials locally so Terraform can use them:
   ```text
   gcloud auth application-default login
   ```

3. Create a working directory for your Terraform files and change to it:
   ```text
   mkdir example-directory
   cd example-directory
   ```

4. Save environment variables containing the following credentials:
   
   * your Metal auth token or Equinix account API token
   * your Metal project ID
   * the GCP project ID for project you specified or created in step 1
  
    ```text
    export TF_VAR_metal_auth_token=abcd1e23-a1bc-1234-abcd-1a2bcde34fa5
    export TF_VAR_metal_project_id=example-metal-project-id
    export TF_VAR_gcp_project_id=example-anthos-gcp-project"
    ```


5.  Create the Terraform configuration file called `main.tf` shown in the example and add the following key-value pairs to it, specifying your own values:
   
    * The **metal_create_project** key with the value set to `false`
    * The **ha_control_plane** key with the value set to `false` if your plan's maximum number of servers is below 6, and `true` if your plan's maximum is above 6. Disabling the HA control plane reduces the node requirements of the deployment at the expense of resilience. 
    * The **facility** key with your desired server location
    * The **cp_plan** key with the server type you want to use for control plane nodes, this must be a minimum of c3.medium.x86
    * The **worker_plan** key with the server type you want to use for worker nodes, this must be a minimum of c3.medium.x86
    * The **worker_count** key with the number of worker nodes you want in your cluster, this must be a minimum of 3
    * The **storage_module** key with the `portworx` value
    * Optionally, you can create a **storage_options** list containing the following key-value pairs:
        * The **portworx_version** key with a version of Portworx you want to deploy. If you do not specify a version, Terraform will apply the latest available version of Portworx.
        * The **portworx_license** key with a Portworx license key to activate when Portworx deploys. If you do not specify a portworx license, Portworx will deploy and start in trial mode. 
        
            ```text
            # variables.tf
            variable "metal_auth_token" {}
            variable "metal_project_id" {}
            variable "gcp_project_id" {}

            # main.tf
            module "anthos-on-baremetal" {
              source  = "equinix/anthos-on-baremetal/metal" 
              version = "0.5.1"
             
              gcp_project_id = var.gcp_project_id
              metal_auth_token = var.metal_auth_token
              metal_project_id = var.metal_project_id
              metal_create_project = false 
              ha_control_plane = true 
              facility = "da11"
              cp_plan = "c3.medium.x86"
              worker_plan = "c3.medium.x86" 
              worker_count = 3
              storage_module = "portworx"
              storage_options = {
                portworx_version = "2.7"
                portworx_license = "abc-example-key-123"
              }
            }
            ```

    {{<info>}}
**NOTE:** For reference information on other variables you can modify to further customize your Portworx deployment, see the [Available variables](https://github.com/equinix/terraform-metal-anthos-on-baremetal#available-variables) section of the project's GitHub Readme.
    {{</info>}}

1.  Initialize Terraform in your working directory and apply the configuration file you just created to deploy your cluster:
    ```text
    terraform init
    terraform apply
    ```

{{<info>}}
**NOTE:** To make changes to your deployment in the future, change the `main.tf` configuration file and reapply it. 
{{</info>}}