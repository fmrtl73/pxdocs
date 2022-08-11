---
title: Kubernetes Secrets
logo: /logos/other.png
keywords: Kubernetes Secrets, k8s, encryption keys, secrets, Volume Encryption, Cloud Credentials, secret store
description: Instructions on using Kubernetes secrets with Portworx
weight: 400
disableprevnext: true
series: key-management
noicon: true
aliases:
    - /key-management/kubernetes-secrets/
---
Portworx can integrate with Kubernetes Secrets to store your encryption keys/secrets and credentials. Follow the instructions on this page to configure Portworx with Kubernetes Secrets. Kubernetes Secrets can then be used to store Portworx secrets for Volume Encryption and Cloud Credentials.

## Configuring Kubernetes Secrets with Portworx

### New installation

When generating the Portworx Kubernetes spec file on the Portworxspec generator page in [PX-Central](https://central.portworx.com), select `Kubernetes` from the `Secrets Store Type` list under `Advanced Settings`. For more details on how to generate the Portworx spec for Kubernetes, [click here](/install-portworx/).

### Existing installation

Previously, Portworx stored credentials/secrets in a Kubernetes namespace called `portworx`. Only if you upgrade Portworx, edit the DemonSet to use Kubernetes secrets as explained in the following sections.

#### Permissions to access secrets
When you have upgraded Portworx as explained in the _Kubernetes_ section under _Upgrades_ in the _Reference_ topic, you do not have to create the following namespace and roles. If the following objects are missing, create them using `kubectl`:

```text
cat <<EOF | kubectl apply -f -
# Namespace to store credentials
apiVersion: v1
kind: Namespace
metadata:
  name: portworx
---
# Role to access secrets under portworx namespace only
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role
  namespace: portworx
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "create", "update", "patch"]
---
# Allow portworx service account to access the secrets under the portworx namespace
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role-binding
  namespace: portworx
subjects:
- kind: ServiceAccount
  name: px-account
  namespace: kube-system
roleRef:
  kind: Role
  name: px-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

#### Edit the Portworx DaemonSet

You must edit the Portworx DaemonSet to use Kubernetes secrets, so that all the new Portworx nodes start using Kubernetes secrets.

```text
kubectl edit daemonset portworx -n kube-system
```

Add the `"-secret_type", "k8s"` arguments to the `portworx` container in the daemonset. It should look something like this:

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
    - k8s
    name: portworx
```

Editing the DaemonSet also restarts all the Portworx pods.

## Creating secrets with Kubernetes

The following section describes the key generation process with Portworx and Kubernetes which can be used for encrypting volumes.

### Setting cluster wide secret key

A cluster wide secret key is a common key that can be used to encrypt all your volumes. Create a cluster wide secret in Kubernetes using `kubectl` command. Use the same namespace on which you've installed Portworx, potentially, `kube-system`:

```text
NAMESPACE=kube-system
kubectl -n ${NAMESPACE} create secret generic px-vol-encryption \
  --from-literal=cluster-wide-secret-key=<value>
```

The cluster wide secret resides in the `px-vol-encryption` secret under the `kube-system` namespace. 

Provide the cluster wide secret key to Portworx, that acts as the default encryption key for all volumes.

```text
PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n ${NAMESPACE} -- /opt/pwx/bin/pxctl secrets set-cluster-key \
  --secret cluster-wide-secret-key
```
{{<info>}}
**Note:** The cluster wide key is the secret name where the value to encrypt key exists and does not contain the value to encrypt.
{{</info>}}

Run this command only once for the cluster. If you have added the cluster secret key through _config.json_, the above command overwrites it. Even on subsequent Portworx restarts, the cluster secret key in _config.json_ is ignored for the one set through the CLI.

### (Optional) Authenticating with Kubernetes Secrets using the Portworx CLI

Verify Kubernetes secrets and authenticate Portworx with Kubernetes Secrets using the Portworx CLI, by running the following command:

```text
PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n ${NAMESPACE} -- /opt/pwx/bin/pxctl secrets k8s login
```
{{<info>}}
**Important:** You need to run this command on all your Portworx nodes, so that you could create and mount encrypted volumes on all nodes.
{{</info>}}

If the CLI is used to authenticate with Kubernetes Secrets, for every restart of the Portworx container it needs to be re-authenticated with Kubernetes Secrets by running the `k8s login` command on that node.


## Using Kubernetes Secrets with Portworx

{{<homelist series="kubernetes-secret-uses">}}
