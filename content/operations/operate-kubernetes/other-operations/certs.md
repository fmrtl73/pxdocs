---
title: "Certificates as Kubernetes Secrets"
keywords: SSL certificates, certs, kubernetes secrets, k8s
description: Certificates as Kubernetes secrets.
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/other-operations/certs/
---
Sometimes it is necessary to store an SSL certificate as a Kubernetes secret. The example here is to secure a third-party S3-compatible objectstore for use with Portworx.

### 1. Create the secret

* Copy your certificate to where the `kubectl` is configured for this Kubernetes cluster. Copy the `objectstore.pem` file to the `/opt/certs` folder.

* Create the secret:

```text
kubectl -n kube-system create secret generic objectstore-cert --from-file=/opt/certs/
```

* (Optional) Confirm if the secret was created correctly:

```text
kubectl -n kube-system describe secret objectstore-cert
```

### 2. Provide secret to Portworx

Based on your Portworx installation type, provide secret to Portworx by performing one of the following steps (2a or 2b). 

### 2a. Portworx DaemonSet

* Update the Portworx DaemonSet to mount the secret and the environment variable:

```text
kubectl -n kube-system edit ds portworx
```

The `volumeMounts:` section in the DaemonSet will have:

```text
volumeMounts:
  - mountPath: /etc/pwx/objectstore-cert
    name: objectstore-cert
```

The `volumes:` section in the DaemonSet will have:

```text
volumes:
  - name: objectstore-cert
    secret:
      secretName: objectstore-cert
      items:
      - key: objectstore.pem
        path: objectstore.pem
```

The `env:` section in the DaemonSet will have:

```text
env:
  - name: "AWS_CA_BUNDLE"
    value: "/etc/pwx/objectstore-cert/objectstore.pem"
```

*  After saving the modified DaemonSet, Portworx will restart in a rolling update.

### 2b. Portworx operator

 * Update the Portworx `StorageCluster` to mount the secret and the environment variable:

```text
   kubectl -n kube-system edit storagecluster portworx
```

```text
   spec:
     volumes:
     - name: objectstore-cert
       mountPath: /etc/pwx/objectstore-cert
       secret:
         secretName: objectstore-cert
         items:
         - key: objectstore.pem
           path: objectstore.pem
     env:
     - name: "AWS_CA_BUNDLE"
       value: "/etc/pwx/objectstore-cert/objectstore.pem"
```

* After saving the modified `StorageCluster`, Portworx will restart in a rolling update. 
