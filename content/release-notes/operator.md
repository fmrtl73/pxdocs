---
layout: page
title: "Operator Release Notes"
description: Release notes for the Portworx Operator.
keywords: portworx, release notes, operator
weight: 35000
series: release-notes
aliases:
    - /reference/release-notes/operator
---

## 1.8.0

Apr 14, 2022

### Updates

* Daemonset to operator migration is in Beta release.
* Added support for passing custom labels to Portworx API service from StorageCluster.
* Operator now enables the Autopilot component to communicate securely using tokens when PX-Security is enabled in the Portworx cluster.
* Added field `preserveFullCustomImageRegistry` in StorageCluster spec to preserve full image path when using custom image registry.
* Operator now retrieves the version manifest through proxy if `PX_HTTP_PROXY` is configured.
* Stork, Stork scheduler, CSI, and PVC controller pods are now deployed with [`topologySpreadConstraints`](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/) to distribute pod replicas across Kubernetes failure domains.
* Added support for installing health monitoring sidecars from StorageCluster.
* Added support for installing snapshot controller and CRD from StorageCluster.
* The feature gate for CSI is now deprecated and replaced by setting `spec.csi.enabled` in StorageCluster.
* Added support to enable hostPID to Portworx pods using the annotation `portworx.io/host-pid="true"` in StorageCluster.
* Operator now sets `fsGroupPolicy` in the CSIDriver object to `File`. Previously it was not set explicitly, and the default value was `ReadWriteOnceWithFsType`.
* Added `skip-resource` annotation to PX-Security Kubernetes secrets to skip backing them to the cloud.
* Operator now sets the dnsPolicy of Portworx pod to `ClusterFirstWithHostNet` by default.
* When using Cloud Storage, Operator validates that the node groups in StorageCluster use only one common label selector key across all node groups. It also validates that the value matches `spec.cloudStorage.nodePoolLabel` if a is present. If the value is not present, it automatically populates it with the value of the common label selector.

### Bug Fixes

* Fixed Pod Disruption Budget issue blocking Openshift upgrade on Metro DR setup.
* Fixed Stork scheduler's pod anti-affinity by adding the label `name: stork-scheduler` to Stork scheduler deployments.
* When a node level spec specifies a cloud storage configuration, we no longer set the cluster level default storage configuration. Before this fix, the node level cloud storage configuration would be overwritten.
