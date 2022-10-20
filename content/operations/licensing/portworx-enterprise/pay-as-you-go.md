---
title: Switch to pay-as-you-go billing on an existing Kubernetes cluster
linkTitle: Enable pay-as-you-go billing
keywords: pay-as-you-go, PAYG, 
description: Learn how you can enable pay-as-you-go billing for Portworx.
weight: 300 
aliases:
  - /reference/knowledge-base/licensing/pay-as-you-go/
  - /reference/licensing/pay-as-you-go/
---

If you have already set up your cluster using any of the paid or free Portworx licenses, you can switch to pay-as-you-go billing by acquiring a pay-as-you-go account key from the Pure Storage support team and performing the following steps.

{{<info>}}
**NOTE:** For enabling pay-as-you-go billing on your air-gapped cluster, refer to [this page](/operations/licensing/portworx-enterprise/pay-as-you-go-air-gapped).
{{</info>}}

## Prerequisites

* A pay-as-you-go account key, acquired by contacting the Pure Storage support team.

## Enable pay-as-you-go billing

1. Create a Kubernetes secret and place your pay-as-you-go account key into it:
   
    ```text
    kubectl create secret generic px-saas-key --from-literal=account-key=<PAY-AS-YOU-GO-KEY> -n kube-system
    ```

2. Patch the pay-as-you-go secret into your cluster. Follow the steps appropriate for your cluster's install method:

    * **Operator-based installations:** 
    
        1. Add the following spec to your StorageCluster object:

            ```text
            env:
            - name: "SAAS_ACCOUNT_KEY_STRING"
              valueFrom:
                secretKeyRef:
                  name: px-saas-key
                  key: account-key
            ```

        2. Patch the `stc` with:
            
            ```text
            stc=$(kubectl get stc -n portworx -o jsonpath='{.items[0].metadata.name}')
            ```

        3. Patch the stc using the secret you created in step 1:

            ```text
            kubectl  patch stc $stc  --type='json' -p='[{"op": "add", "path": "/spec/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n kube-system
            ```

    * **Daemonset-based installations:**
		
        Patch the daemonset using the secret you created in step 1:

        ```text
        kubectl patch ds portworx --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n kube-system
        ```

  {{<info>}}
   **NOTE:** If you change the value of your pay-as-you-go key secret in the future, you must perform a rolling update of your Portworx pods to propagate the change. 
   {{</info>}}


   
