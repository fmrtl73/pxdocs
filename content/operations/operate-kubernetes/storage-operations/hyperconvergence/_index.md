---
title: Run Hyper Converged Using Stork
weight: 200
keywords: hyper-convergence, stork, kubernetes, k8s
description: Learn how to use Stork to run your applications hyperconverged with their data
noicon: true
series: k8s-storage
aliases:
    - /portworx-install-with-kubernetes/storage-operations/hyperconvergence/
---
### Hyper-convergence

When a pod runs on the same host as its volume, it is known as convergence or hyper-convergence. Because this configuration reduces the network overhead of an application, performance is typically better.

### Using scheduler convergence

The recommended method to run your pods hyperconverged is to use [Stork](/operations/operate-kubernetes/storage-operations/stork).

Once you have installed Stork, the webhook controller is enabled by default and will use Stork as the scheduler. If you have disabled `webhook-controller` with the flag, all you need to do is add `schedulerName: stork` in your application specs. Stork will then ensure that the nodes with data for a volume get prioritized when pods are being scheduled.

For example, this is how you would specify the scheduler name in a MySQL deployment:

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
        version: "1"
    spec:
      schedulerName: stork
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-data
```

{{<info>}}
**NOTE:** You can force a Pod to be scheduled on the same node as a replica by specifying `stork.libopenstorage.org/preferLocalNodeOnly: "true"` in the `spec.template.metadata.annotations` section of your `StatefulSet` object.
{{</info>}}


### Disabling scheduler convergence
You can disable Stork's hyperconverged pod prioritization with the `stork.libopenstorage.org/disableHyperconvergence: "true"` pod annotation.
