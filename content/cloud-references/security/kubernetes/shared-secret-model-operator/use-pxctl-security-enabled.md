---
title: "Use pxctl with security enabled"
keywords: generate, token, jwt, pxctl, authorization, security, pxctl, context
weight: 80
series: ra-shared-secrets-model
---

Once a storage cluster with security enabled is running, a cluster admin must set up a [pxctl context](/reference/cli/authorization/) on each node in order to interact with the system.

The following steps will guide an operator-based storage admin to setup pxctl contexts on each node:

1. Retrieve the admin token from the namespace in which Portworx was installed and store it in a variable `ADMIN_TOKEN`:

    ```text
    ADMIN_TOKEN=$(kubectl -n kube-system get \
      secret px-admin-token -o json \
    | jq -r '.data."auth-token"' \
    | base64 -d)
    ```

2. Find the portworx pod which is running on the node in which the admin wants to interact with:

    ```
    kubectl -n kube-system get pods -l name=portworx -o wide
    ```

    Copy the desired portworx pod for the next command.

3. Save the admin token in the `pxctl` [context](/reference/cli/authorization/#contexts) for that pod:

    ```text
    kubectl -n kube-system exec -ti <portworx_pod> -- /opt/pwx/bin/pxctl context create admin --token=$ADMIN_TOKEN
    ```

4. Exec into the portworx container to perform any pxctl commands:

    ```
    kubectl -n kube-system exec -ti <portworx_pod> /bin/bash

    ```
{{<info>}}
**Note:**
This pxctl context will need to be refreshed every time the token expires. This is 24 hours by default, but can be extended. See the [customizing security](/cloud-references/security/kubernetes/shared-secret-model-operator/customizing-security/) page for more information.
{{</info>}}
    