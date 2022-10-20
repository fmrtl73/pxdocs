---
title: Monitor your Portworx cluster
weight: 1000
description: Set up monitoring
keywords: Portworx, containers, storage
---

You can use the following technologies to monitor your Portworx cluster:

* Prometheus to collect data
* Alertmanager to provide notifications
* Grafana to visualize your data

### Verify monitoring using Prometheus

You can monitor your Portworx cluster using Prometheus. Portworx deploys Prometheus by default, but you can verify the deployment:

1. Verify that Prometheus pods are running by entering the following `kubectl get pods` command:

    ```text 
    kubectl get pods -A | grep -i prometheus
    ```
    ```output
    kube-system   prometheus-px-prometheus-0                              2/2     Running            0                59m
    kube-system   px-prometheus-operator-59b98b5897-9nwfv                 1/1     Running            0                60m
    ```

2. Verify that the Prometheus `px-prometheus` and `prometheus operated` services exist by entering the following `kubectl get service` command:

    ```text 
    kubectl -n kube-system  get service | grep -i prometheus
    ```
    ```output
    prometheus-operated         ClusterIP   None             <none>        9090/TCP                       63m
    px-prometheus               ClusterIP   10.99.61.133     <none>        9090/TCP                       63m
    ```
### Set up Alertmanager

Prometheus Alertmanager handles alerts sent from the Prometheus server based on rules you set. If any Prometheus rule is triggered, Alertmanager sends a corresponding notification to the specified receivers. You can configure these receivers using an Alertmanager config file. Perform the following steps to configure and enable Alertmanager: 

1. Create a valid [Alertmanager configuration](https://prometheus.io/docs/alerting/latest/configuration/) file and name it `alertmanager.yaml`. The following is a sample for Alertmanager, the settings used in your environment may be different:

    ```text
    global:
      # The smarthost and SMTP sender used for mail notifications.
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'abc@test.com'
      smtp_auth_username: "abc@test.com"
      smtp_auth_password: 'xyxsy'
    route:
      group_by: [Alertname]
      # Send all notifications to me.
      receiver: email-me
    receivers:
    - name: email-me
      email_configs:
      - to: abc@test.com
        from: abc@test.com
        smarthost: smtp.gmail.com:587
        auth_username: "abc@test.com"
        auth_identity: "abc@test.com"
        auth_password: "abc@test.com"
    ```

1. Create a secret called `alertmanager-portworx` in the same namespace as your StorageCluster object:

    ```text
    kubectl -n kube-system create secret generic alertmanager-portworx --from-file=alertmanager.yaml=alertmanager.yaml
    ```

2. Edit your StorageCluster object to enable Alertmanager:

    ```text
    kubectl -n kube-system edit stc <px-cluster-name>
    ```

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    monitoring:
        prometheus:
          enabled: true
          exportMetrics: true
          alertManager:
            enabled: true
    ```

3. Verify the Alertmanager pods are running by executing the following command:

```text
kubectl -n kube-system get pods | grep -i alertmanager
```
```output
alertmanager-portworx-0                    2/2     Running   0              4m9s
alertmanager-portworx-1                    2/2     Running   0              4m9s
alertmanager-portworx-2                    2/2     Running   0              4m9s
```

5. Access Alertmanager by setting up port forwarding and browsing to the specified port. In this example, port forwarding is provided for ease of access to the Alertmanager service from your local machine using the port 9093:

    ```text 
    kubectl -n kube-system port-forward service/alertmanager-portworx 9093:9093
    ```
{{<info>}}
**NOTE:** PX-Central on-premises includes Grafana and Portworx dashboards natively, which you can use to monitor your Portworx cluster. Refer to the [PX-Central documentation](https://central.docs.portworx.com/) for further details.
{{</info>}}

### Configure Grafana

You can connect to Prometheus using Grafana to visualize your data. 

1. Enter the following `curl` commands to download the Grafana dashboard and datasource configuration files:

    ```text 
    curl -O https://docs.portworx.com/samples/k8s/pxc/grafana-dashboard-config.yaml
    ```
    ```output
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   211  100   211    0     0    596      0 --:--:-- --:--:-- --:--:--   596
    ```

    ```text 
    curl -O https://docs.portworx.com/samples/k8s/pxc/grafana-datasource.yaml
    ```
    ```output
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100  1625  100  1625    0     0   4456      0 --:--:-- --:--:-- --:--:--  4464
    ```

2. Create a configmap for the dashboard and data source.
    
    ```text 
    kubectl -n kube-system create configmap grafana-dashboard-config --from-file=grafana-dashboard-config.yaml
    ```
    ```text
    kubectl -n kube-system create configmap grafana-source-config --from-file=grafana-datasource.yaml
    ```

3. Download and install Grafana dashboards using the following `curl` and `kubectl` commands:

    ```text
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-cluster-dashboard.json" -o portworx-cluster-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-node-dashboard.json" -o portworx-node-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-volume-dashboard.json" -o portworx-volume-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-performance-dashboard.json" -o portworx-performance-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-etcd-dashboard.json" -o portworx-etcd-dashboard.json
    ```

    ```text
    kubectl -n kube-system create configmap grafana-dashboards \
    --from-file=portworx-cluster-dashboard.json \
    --from-file=portworx-performance-dashboard.json \
    --from-file=portworx-node-dashboard.json \
    --from-file=portworx-volume-dashboard.json \
    --from-file=portworx-etcd-dashboard.json 
    ```

4. Enter the following `kubectl apply` command to download and install Grafana YAML file:

    ```text 
    kubectl apply -f https://docs.portworx.com/samples/k8s/pxc/grafana.yaml
    ```

5. Verify if grafana pod is running by running the following `kubectl` command:

    ```text
    kubectl -n kube-system get pods | grep -i grafana
    ```
    ```output
    grafana-7d789d5cf9-bklf2                   1/1     Running   0              3m12s
    ```

6. Access Grafana by setting up port forwarding and browsing to the specified port. In this example, port forwarding is provided for ease of access to the Grafana service from your local machine using the port 3000:

    ```text
    kubectl -n kube-system port-forward service/grafana 3000:3000
    ```

7. Navigate to Grafana by browsing to `http://localhost:3000`.
    
8. Enter the default credentials to login:

    * **login:**  `admin`
    * **password:** `admin`
