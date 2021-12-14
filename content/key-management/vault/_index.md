---
title: Vault
logo: /logos/hashicorp-vault.png
keywords: Vault Key Management, Hashicorp, encryption keys, secrets, Volume Encryption, Cloud Credentials, secret store, passwords
description: Instructions on using Vault key management with Portworx
weight: 5
disableprevnext: true
series: key-management
noicon: true
---

Portworx can integrate with Vault to store your encryption keys, secrets, and credentials. This topic explains how to connect a Portworx cluster to a Vault development server endpoint and use it to store secrets that you can use for encrypting volumes.

<!-- Since we only provide instructions for setting up a Vault development environment, we should mention that somewhere. -->

## Set up Vault

Set up and deploy Vault by following the instructions in the [Install Vault section](https://www.vaultproject.io/docs/install) of the Vault documentation. This includes installation, setting up policies, and configuring secrets.

{{<info>}}
**NOTE:** To run a dev server, use the `vault server -dev` command. This will only run on 127.0.0.1:8200, and cannot be connected by the container. Ensure the server endpoint is securely exposed to the Portworx clusters. 
{{</info>}}

## Set up the Vault development environment

Once you've set up Vault, you're ready to set up your development environment. 

{{<info>}}
**NOTE:** Pure Storage does not recommend using this for production environments.
{{</info>}}

1. Create a `config.hcl` file, then start the Vault server:

    `config.hcl`
    ```text
    listener "tcp" {
      address = "0.0.0.0:8200"
      tls_disable = 1
    }

    storage "file" {
      path = "/tmp/vault-data"
    }

    disable_mlock = true
    ```
    ```text
    mkdir -p /tmp/vault-data
    vault server -config=config.hcl
    ```

2. When Vault initializes, it will present the unseal keys and initial root token. Securely store and distribute the keys, as they will be used in later operations.

    ```text
    export VAULT_ADDR='http://127.0.0.1:8200'
    vault operator init
    ```
    ```output
    Unseal Key 1: 4jYbl2CBIv6SpkKj6Hos9iD32k5RfGkLzlosrrq/JgOm
    Unseal Key 2: B05G1DRtfYckFV5BbdBvXq0wkK5HFqB9g2jcDmNfTQiS
    Unseal Key 3: Arig0N9rN9ezkTRo7qTB7gsIZDaonOcc53EHo83F5chA
    Unseal Key 4: 0cZE0C/gEk3YHaKjIWxhyyfs8REhqkRW/CSXTnmTilv+
    Unseal Key 5: fYhZOseRgzxmJCmIqUdxEm9C3jB5Q27AowER9w4FC2Ck

    Initial Root Token: s.KkNJYWF5g0pomcCLEmDdOVCW

    Vault initialized with 5 key shares and a key threshold of 3. Please securely
    distribute the key shares printed above. When the Vault is re-sealed,
    restarted, or stopped, you must supply at least 3 of these keys to unseal it
    before it can start servicing requests.

    Vault does not store the generated master key. Without at least 3 key to
    reconstruct the master key, Vault will remain permanently sealed!

    It is possible to generate new unseal keys, provided you have a quorum of
    existing unseal keys shares. See "vault operator rekey" for more information.
    ```

3. When you first start Vault, you must unseal it. Enter the following command to unseal the Vault server, repeat it 3 times: 

    ```text
    vault operator unseal
    ```
    ```output
    Unseal Key (will be hidden):
    Key                Value
    ---                -----
    Seal Type          shamir
    Initialized        true
    Sealed             true
    Total Shares       5
    Threshold          3
    Unseal Progress    1/3
    Unseal Nonce       d3d06528-aafd-c63d-a93c-e63ddb34b2a9
    Version            1.7.0
    Storage Type       raft
    HA Enabled         true
    ```

4. Log in to the Vault server using the root token you generated in step 2:

    ```text
    vault login <initial-root-token>
    ```

5. Verify the installation by entering the following vault command, specify your own value for `<my-vault-secret>`:

    ```text 
    vault kv put secret/my-secret my-value=<my-vault-secret> 
    ```
    ```output
    Key              Value
    ---              -----
    created_time     2019-06-19T17:20:22.985303Z
    deletion_time    n/a
    destroyed        false
    version          1
    ```

## Kubernetes users

### Step 1: Choose the Vault authentication method.

  Authentication methods are responsible for authenticating Portworx with Vault. Based on your Vault configuration and the authentication method you choose, you must use one of the following two methods:

  * **Using Token authetication:** A static Vault token is provided to Portworx.
  * **Using Kubernetes authentication:** Portworx uses Kubernetes service account to fetch and refresh Vault tokens.

#### Using token authentication method

With this method, Portworx requires a Vault static token that you should provide through a Kubernetes secret.

   * Provide Vault credentials to Portworx. Refer to the [Vault credentials reference](#vault-credentials-reference) for details on the credentials. 

     Portworx reads the Vault credentials required to authenticate with Vault through a Kubernetes secret. Create a Kubernetes secret with the name `px-vault` in the `portworx` namespace. Following is an example Kubernetes secret spec:

    ```text
    apiVersion: v1
    kind: Secret
    metadata:
      name: px-vault
      namespace: portworx
    type: Opaque
    data:
      VAULT_ADDR: (required)<base64 encoded value of the vault endpoint address>
      VAULT_TOKEN: (required)<base64 encoded value of the vault token>
      VAULT_CACERT: (recommended)<base64 encoded file path where the CA Certificate is present on all the nodes>
      VAULT_CAPATH: (recommended)<base64 encoded file path where the Certificate Authority is present on all the nodes>
      VAULT_CLIENT_CERT: (recommended)<base64 encoded file path where the Client Certificate is present on all the nodes>
      VAULT_CLIENT_KEY: (recommended)<base64 encoded file path where the Client Key is present on all the nodes>
      VAULT_TLS_SERVER_NAME: (recommended)<base64 encoded value of the TLS server name>
      VAULT_BACKEND_PATH: (optional)<base64 encoded value of the custom backend path if different than the default "secret">
      VAULT_NAMESPACE: (optional)<base64 encoded value of the global vault namespace for portworx>
    ```

Portworx searches for this secret with name `px-vault` under the `portworx` namespace.

{{<info>}}
**NOTE:**

* If the `VAULT_TOKEN` provided in the secret above is refreshed, then you must manually update this secret.
* If you installed Portworx using the Operator, then you must create the Kubernetes secret in the same namespace in which you deployed Portworx.
{{</info>}}

After configuring Vault using the Vault authentication method, proceed to [Step 2](#step-2-setup-vault-as-the-secrets-provider-for-portworx).

#### Using Kubernetes authentication method

This method allows Portworx to authenticate with Vault using a Kubernetes service account token. For more information about how to setup Kubernetes Vault authentication, refer to the [Vault documentation](https://www.vaultproject.io/docs/auth/kubernetes).

  * Create a `ServiceAccount` for Vault authentication delegation.

    Run the following `kubectl create` commands to create a `ServiceAccount` and `ClusterRoleBinding`. Vault uses this `ServiceAccount` and its associated token to authenticate requests from Portworx. Vault uses the [Kubernetes TokenReview API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#tokenreview-v1-authentication-k8s-io).

    <!-- I had to bump this to 1.19, as support for the previous version was dropped. Can we just use the latest for this and drop the version altogether? -->

```text
kubectl create serviceaccount vault-auth -n kube-system
kubectl create clusterrolebinding vault-tokenreview-binding --clusterrole=system:auth-delegator --serviceaccount=kube-system:vault-auth
```

  * Enable Kubernetes authentication in Vault. Enter the following `vault auth` command to enable Kubernetes authentication in Vault:

```text
vault auth enable kubernetes
```

  * Create a Kubernetes authentication configuration in Vault. Enter the following `export` commands to get the JWT token and CA certificate of Kubernetes `ServiceAccount`:

```text
export VAULT_SA_NAME=$(kubectl get sa vault-auth -n kube-system \
     -o jsonpath="{.secrets[*]['name']}")

export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -n kube-system \
     -o jsonpath="{.data.token}" | base64 --decode; echo)

export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -n kube-system \
     -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
```

    Enter the following `vault write` command, replacing `<kubernetes-endpoint>` with your kubernetes `api-server` endpoint to write a Kubernetes authentication configuration to Vault:

```text
vault write auth/kubernetes/config \
        token_reviewer_jwt="$SA_JWT_TOKEN" \
        kubernetes_host="<kubernetes-endpoint>" \
        kubernetes_ca_cert="$SA_CA_CRT"
```

  * Create a Kubernetes authentication role for Portworx, named `portworx`, in Vault:

```text
vault write auth/kubernetes/role/portworx \
        bound_service_account_names=px-account \
        bound_service_account_namespaces=kube-system \
        policies=portworx \
        ttl=<ttl>
```

    {{<info>}}
**NOTE:** If you deployed Portworx using the Operator, then you must enter the following command to create a Kubernetes authentication role, replacing `<namespace>` with the namespace in which you deployed Portworx:

```text
vault write auth/kubernetes/role/portworx \
        bound_service_account_names=portworx \
        bound_service_account_namespaces=<namespace> \
        policies=portworx \
        ttl=<ttl>
```
    {{</info>}}

  * Provide Vault credentials to Portworx. Refer to [Vault credentials reference](#vault-credentials-reference) for details on the credentials. 

    Portworx reads the Vault credentials required to authenticate with Vault through a Kubernetes secret. Create a Kubernetes secret with the name `px-vault` in the `portworx` namespace. Refer to the following example Kubernetes secret specification to create your own secret:

    ```text
    apiVersion: v1
    kind: Secret
    metadata:
      name: px-vault
      namespace: portworx
    type: Opaque
    data:
      VAULT_ADDR: <base64 encoded value of the vault endpoint address>
      VAULT_BACKEND_PATH: <base64 encoded value of the custom backend path if different than the default "secret">
      VAULT_CACERT: <base64 encoded file path where the CA Certificate is present on all the nodes>
      VAULT_CAPATH: <base64 encoded file path where the Certificate Authority is present on all the nodes>
      VAULT_CLIENT_CERT: <base64 encoded file path where the Client Certificate is present on all the nodes>
      VAULT_CLIENT_KEY: <base64 encoded file path where the Client Key is present on all the nodes>
      VAULT_TLS_SERVER_NAME: <base64 encoded value of the TLS server name>
      VAULT_AUTH_METHOD: a3ViZXJuZXRlcw== // base64 encoded value of "kubernetes"
      VAULT_AUTH_KUBERNETES_ROLE: cG9ydHdvcng= // base64 encoded value of the kubernetes auth role "portworx"
      VAULT_NAMESPACE: <base64 encoded value of the global vault namespace for portworx>
    ```

    {{<info>}}
  **NOTE:** Set the value of `VAULT_AUTH_KUBERNETES_ROLE` to the `base64` encoded value of the role created in the previous step.
    {{</info>}}

    Portworx searches for this secret with name `px-vault` under the `portworx` namespace. While installing Portworx, it creates a Kubernetes role binding that grants access to reading Kubernetes secrets only from the `portworx` namespace.

    {{<info>}}
**NOTE:** If you installed Portworx using the Operator, then you must create the Kubernetes secret in the same namespace in which you deployed Portworx.
    {{</info>}}

### Step 2: Setup Vault as the secrets provider for Portworx

#### New Installation

When generating the [Portworx Kubernetes specification file](https://central.portworx.com), select `Vault` from the "Secrets type" list.

#### Existing Installation

For an existing Portworx cluster, perform the following step to configure Vault as the secrets provider:

  * Edit the Portworx DaemonSet `secret_type` field to `vault`, so that all the new Portworx nodes start using Vault:

```text
kubectl edit daemonset portworx -n kube-system
```

    Add the `"-secret_type", "vault"` arguments to the `portworx` container in the DaemonSet. It should look something like this:

```text
containers:
  - args:
    - -c
    - testclusterid
    - -s
    - /dev/sdb
    - -x
    - kubernetes
    - -secret_type
    - vault
    name: portworx
```

    Editing the DaemonSet will restart all the Portworx pods.

    {{<info>}}
**NOTE:** If you installed Portworx using the Operator, then you must edit your `StorageCluster` object, setting the value of the `specs.secretsProvider` field to `vault`.
    {{</info>}}

## Other users {#other-users}

1. Provide Vault credentials to Portworx.

    Provide the following Vault credentials (key value pairs) as environment variables to Portworx:

    * (Required) `VAULT_ADDR=`vault endpoint address
    * (Required) `VAULT_TOKEN=`vault token
    * (Optional) `VAULT_BASE_PATH=`vault base path>
    * (Optional) `VAULT_BACKEND_PATH=`custom backend path if different than the default `secret`
    * (Optional) `VAULT_CACERT=`file path where the CA Certificate is present on all the nodes
    * (Optional) `VAULT_CAPATH=`file path where the Certificate Authority is present on all the nodes
    * (Optional) `VAULT_CLIENT_CERT=`file path where the Client Certificate is present on all the nodes
    * (Optional) `VAULT_CLIENT_KEY=`file path where the Client Key is present on all the nodes
    * (Optional) `VAULT_TLS_SERVER_NAME=`TLS server name

    {{<info>}}
**NOTE:** Providing `VAULT_NAMESPACE` as environment variable is not supported. Please specify the vault namespace through `pxctl` commands as shown in subsequent sections.
    {{</info>}}

2. Set up Vault as the secrets provider for Portworx.

#### New installation

While installing Portworx set the input argument `-secret_type` to `vault`.

#### Existing installation

Based on your installation method, provide the `-secret_type vault` input argument and restart Portworx on all the nodes.

## Vault security policies

If you configured Vault strictly with policies, then the Vault token provided to Portworx should follow one of the following policies:

 ```text
 # Read and List capabilities on mount to determine which version of kv backend is supported
 path "sys/mounts/*"
 {
 capabilities = ["read", "list"]
 }

 # V1 backends (Using default backend)
 # Provide full access to the portworx subkey
 # Provide -> VAULT_BASE_PATH=portworx to PX (optional)
 path "secret/portworx/*"
 {
 capabilities = ["create", "read", "update", "delete", "list"]
 }

 # V1 backends (Using custom backend)
 # Provide full access to the portworx subkey
 # Provide -> VAULT_BASE_PATH=portworx to PX (optional)
 # Provide -> VAULT_BACKEND_PATH=custom-backend (required)
 path "custom-backend/portworx/*"
 {
 capabilities = ["create", "read", "update", "delete", "list"]
 }


 # V2 backends (Using default backend )
 # Provide full access to the data/portworx subkey
 # Provide -> VAULT_BASE_PATH=portworx to PX (optional)
 path "secret/data/portworx/*"
 {
 capabilities = ["create", "read", "update", "delete", "list"]
 }

 # V2 backends (Using custom backend )
 # Provide full access to the data/portworx subkey
 # Provide -> VAULT_BASE_PATH=portworx to PX (optional)
 # Provide -> VAULT_BACKEND_PATH=custom-backend (required)
 path "custom-backend/data/portworx/*"
 {
 capabilities = ["create", "read", "update", "delete", "list"]
 }
 ```

{{<info>}}
**Note**: Portworx supports only the kv backend of Vault.
{{</info>}}

You can set all the above Vault related fields and the cluster secret key using the Portworx CLI, which is explained in the next section.


## Key generation with Vault

The following sections describe the key generation process with Portworx and Vault, which you can use for encrypting volumes. For more information about encrypted volumes, refer to the [Encrypted volumes using pxctl](/reference/cli/encrypted-volumes) page.

### Setting cluster wide secret key

A cluster-wide secret key is a common key that you can use to encrypt all your volumes. Run the following command to set the cluster secret key:

```text
pxctl secrets set-cluster-key --secret <cluster-wide-secret-key>
```

You should run this command only once for the cluster. If you added the cluster secret key through the `config.json`, then the command above overwrites it. Even on subsequent Portworx restarts, the cluster secret key in `config.json` will be ignored for the one set through the CLI.

## Vault credentials reference

Portworx requires the following Vault credentials to use its APIs:

* **Vault Address [VAULT_ADDR]**

    Vault server address expressed as a URL and port. For example: `https://192.168.11.11:8200`

* **Vault Token [VAULT_TOKEN]**

    Vault authentication token. Refer to the [Vault tokens](https://www.vaultproject.io/docs/concepts/tokens) page for more information about Vault tokens. If you are using the [Kubernetes authentication method](#using-kubernetes-authentication-method) of Vault, then you need not provide the actual token to Portworx.

* **Vault Base Path [VAULT_BASE_PATH]**

    The base path under which Portworx has access to secrets.

* **Vault Backend Path [VAULT_BACKEND_PATH]**

    The custom backend path if different than the default `secret`

* **Vault CA Certificate [VAULT_CACERT]**

    Path to a PEM-encoded CA certificate file that needs to be present on all Portworx nodes. This file is used to verify the SSL certificate of Vault server.
    This variable takes precedence over `VAULT_CAPATH`.

* **Vault CA Path [VAULT_CAPATH]**

    Path to a directory of PEM-encoded CA certificate files that needs to be present on all Portworx nodes.

* **Vault Client Certificate [VAULT_CLIENT_CERT]**

    Path to a PEM-encoded client certificate that needs to be present on all Portworx nodes. This file is used for TLS communication with the Vault server.

* **Vault Client Key [VAULT_CLIENT_KEY]**

    Path to an unencrypted, PEM-encoded private key which corresponds to the matching client certificate. This key file needs to be present on all Portworx nodes.

* **Vault TLS Server Name [VAULT_TLS_SERVER_NAME]**

    Name to use as the SNI host when you connect using TLS.

* **Vault Auth Method [VAULT_AUTH_METHOD]**

    Specifies the authentication method that Portworx should use while authenticating with Vault. "Kubernetes" is the currently supported authenication method.

* **Vault Auth Kubernetes Role [VAULT_AUTH_KUBERNETES_ROLE]**

    Name of the Kubernetes "Auth Role" created in Vault for Portworx. This field is set only when using the Kubernetes authentication method.

* **Vault Namespace [VAULT_NAMESPACE]**

    Allows you to set a global Vault namespace for the Portworx cluster. All Vault requests by Portworx use this Vault namespace, if you do not provide an override.

## Using Vault with Portworx

{{<homelist series="vault-secret-uses">}}
