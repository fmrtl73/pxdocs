---
title: Enabling Pure1 integration for upgrades
weight: 100
keywords: Troubleshoot, diags, phonehome, callhome
description: Find out how to enable the Portworx Pure1 integration for clusters that were created before Portworx 2.8.0
hidden: true
---

This page covers enabling Pure1 integration for following cases

* You had a running Portworx cluster prior to Portworx 2.8.0, and you are now upgrading to 2.8.0 or above
* You installed Portworx 2.8.0 or above but did not enable Pure1 integration and want to do so now

## Pre-requisites

* Portworx 2.8.0 or above
* Outbound access to the internet to allow connection to Pure1

## Operator based install

* Ensure you have running Portworx Operator 1.5.0 or above.
* After you upgrade the Portworx version to 2.8.0 or above in the StorageCluster spec, telemetry will automatically be enabled. This will add the `telemetry` container in Portworx pod.

## DaemonSet based install

This requires 3 steps:

### Step 1. Add the telemetry ConfigMap in your cluster

```text
curl -fsL -o pxtelemetry-config.yaml "https://install.portworx.com/2.8?comp=telemetry"
kubectl apply -f  pxtelemetry-config.yaml
```

### Step 2. Add a `px-role` Role in kube-system namespace

```text
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role
  namespace: kube-system
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: px-role-binding
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: px-account
  namespace: kube-system
roleRef:
  kind: Role
  name: px-role
  apiGroup: rbac.authorization.k8s.io
```

### Step 3. Add the `telemetry` container in the Portworx DaemonSet

This will be a 2-step process. First add the volumes in the pod spec needed for telemetry and then add the telemetry container.

Step a: Add following to the volumes section in the Portworx DaemonSet.

{{<info>}}Note that above is in addition to any existing volumes in your Portworx DaemonSet spec{{</info>}}

```text
      volumes:
        - hostPath:
            path: /var/cache
          name: varcache
        - hostPath:
            path: /etc/timezone
          name: timezone
        - hostPath:
            path: /etc/localtime
          name: localtime
        - configMap:
            items:
              - key: ccm.properties
                path: ccm.properties
              - key: location
                path: location
            name: px-telemetry-config
          name: ccm-config
```

Step b: Add telemetry container to the Portworx DaemonSet

```text
        - name: telemetry
          image: purestorage/ccm-service:3.0.1
          imagePullPolicy: Always
          args:
            ["-Dserver.rest_server.core_pool_size=2"]
          resources:
            limits:
              memory: 512Mi
            requests:
              memory: 256Mi
          env:
            - name: configFile
              value: /etc/ccm/ccm.properties
            - name: PX_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: controller_sn
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
          livenessProbe:
            httpGet:
              host: 127.0.0.1
              path: /1.0/status
              port: 1970
          readinessProbe:
            httpGet:
              host: 127.0.0.1
              path: /1.0/status
              port: 1970
          securityContext:
            privileged: true
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /var/cache
              name: varcache
            - mountPath: /etc/timezone
              name: timezone
            - mountPath: /etc/localtime
              name: localtime
            - mountPath: /etc/ccm
              name: ccm-config
            - mountPath: /var/cores
              name: diagsdump
            - mountPath: /etc/pwx
              name: etcpwx
            - mountPath: /var/run/log
              name: journalmount1
              readOnly: true
            - mountPath: /var/log
              name: journalmount2
              readOnly: true
```