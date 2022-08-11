---
title: Install Alertmanager using the Operator
linkTitle: Alertmanager using Operator
keywords: monitoring, prometheus, alertmanager, Kubernetes, k8s
description: How to use Prometheus Alertmanager for monitoring Portworx on Kubernetes
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/monitoring/alertmanager-operator/
---
Prometheus Alertmanager handles alerts sent from the Prometheus server based on the rules that you setup. If any Prometheus rule is triggered, the alert manager sends a corresponding notification to the specified receivers. You can configure these receivers using an Alertmanager config file.

Perform the following steps to set up an Alertmanager configuration and enable Prometheus and Alertmanager:

1. Create a secret called `alertmanager-portworx` in the same namespace as the StorageCluster object. This secret should contain a valid [Alertmanager configuration](https://prometheus.io/docs/alerting/latest/configuration/). The key for this should be called `alertmanager.yaml`. 

    ```text
    kubectl -n kube-system create secret generic alertmanager-portworx --from-file=alertmanager.yaml
    ```
2. Edit your StorageCluster object to enable Prometheus and Alertmanager:
   
   ```text
   apiVersion: core.libopenstorage.org/v1
   kind: StorageCluster
   metadata:
     name: portworx
     namespace: kube-system
   spec:
     monitoring:
       prometheus:
         enabled: true
         exportMetrics: true
         alertManager:
           enabled: true
    ```

Once applied, you can access Alertmanager by setting up port forwarding and browsing to the specified port:

```text
kubectl -n kube-system port-forward service/alertmanager-portworx 9093:9093
```
