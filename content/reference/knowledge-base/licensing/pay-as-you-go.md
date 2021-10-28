---
title: Switch to pay-as-you-go billing on an existing Kubernetes cluster
linkTitle: Enable pay-as-you-go billing
keywords: pay-as-you-go, PAYG, 
description: Learn how you can enable pay-as-you-go billing for Portworx.
---

If you already set up your cluster using any of the paid or free Portworx licenses (trial, extended, traditional), you can switch to pay-as-you-go billing by acquiring a pay-as-you-go account key from {{<companyName>}} support and performing the steps in this article.

{{<info>}}
**NOTE:** If you're running Portworx on an airgapped cluster that uses an internal license server, you cannot use pay-as-you-go-licenses.
{{</info>}}

## Prerequisites

* A pay-as-you-go account key, acquired by contacting {{<companyName>}} support.

## Enable pay-as-you-go billing

1. Create a Kubernetes secret and place your pay-as-you-go account key into it:
   
    ```text
    kubectl create secret generic px-saas-key --from-literal=account-key=<PAY-AS-YOU-GO-KEY> -n kube-system
    ```

2. Patch the pay-as-you-go secret into your cluster. Follow the steps appropriate for your cluster's install method:

    * **Operator-based installations**
  
        The following spec needs adding to the StorageCluster object:

        ```text
        env:
        - name: "SAAS_ACCOUNT_KEY_STRING"
          valueFrom:
            secretKeyRef:
              name: px-saas-key
              key: account-key
        ```

        Use {{kubectl edit stc}}, or patch with:
        
        ```text
        stc=$(kubectl get stc -n portworx -o jsonpath='{.items[0].metadata.name}')
        ```

        Patch the stc using the secret you created in step 1:

        ```text
        kubectl  patch stc $stc  --type='json' -p='[{"op": "add", "path": "/spec/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n kube-system
        ```

    * **Daemonset-based installations**
		
        Patch the daemonset using the secret you created in step 1:

        ```text
        kubectl patch ds portworx --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n kube-system
        ```

3. (Air gapped only) If you're running Portworx on an air gapped cluster, you must open the firewall to allow communication between cluster nodes and the external license server. 
   
    {{<info>}}
**NOTE:** If your air gapped cluster uses an internal license server, you cannot use pay-as-you-go-licenses.
    {{</info>}}

4. If you're using traditional or extended licenses, contact support to complete the switch to pay-as-you-go billing.

 {{<info>}}
**NOTE:** If you change the value of your pay-as-you-go key secret in the future, you must perform a rolling update of your Portworx pods to propagate the change. 
 {{</info>}}
