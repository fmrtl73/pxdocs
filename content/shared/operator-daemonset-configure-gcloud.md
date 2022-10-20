---
title: configure gcloud shared content
keywords: 
description: configure gcloud shared content
hidden: true
---

### Create your GKE cluster using gcloud

To create a 3-node regional in us-east1 cluster with auto-scaling enabled, run:

<!-- concerns:
* permissions/scopes
* disk size (bump to 128)
* machine type (need more powerful machine)
 -->

    ```text
    gcloud container clusters create px-demo \
        --region us-east1 \
        --node-locations us-east1-b,us-east1-c,us-east1-d \
        --disk-type=pd-ssd \
        --disk-size=128GB \
        --labels=portworx=gke \
        --machine-type=n1-highcpu-8 \
        --num-nodes=3 \
        --image-type ubuntu \
        --scopes compute-rw,storage-ro \
        --enable-autoscaling --max-nodes=6 --min-nodes=3
    ```

### Set your default cluster

After the above command completes, let’s check that everything is properly set up and make this cluster the default cluster while using gcloud:

```text
gcloud config set container/cluster px-demo
gcloud container clusters get-credentials --region <region-name> px-demo
```

Next, we need to open access to the Compute API. Run the following command:

```text
gcloud services enable compute.googleapis.com
```

<!-- we need to understand why we're doing this gcloud stuff -->

### Provide permissions to Portworx 

<!-- This appears to be admin, do we want to constrain this to certain permissions? We want to provide minimum permissions -->

Portworx requires a ClusterRoleBinding for your user to deploy the specs. You can do this using:

```text
kubectl create clusterrolebinding myname-cluster-admin-binding \
    --clusterrole=cluster-admin --user=`gcloud info --format='value(config.account)'`
```