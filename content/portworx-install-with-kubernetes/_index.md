---
title: Portworx on Kubernetes
weight: 2
hidesections: true
description: Documentation on using Portworx in Kubernetes environments
series: top
icon: /img/banner__kubernetes.png
keywords: portworx, kubernetes, k8s, container, storage
---

## Try without installing

You can try Portworx in live interactive tutorials before installing it in your environment.

To view the available playgrounds, continue below.
{{< widelink url="/interactive-tutorials" >}}Interactive tutorials{{</widelink>}}

## Before you begin

Before you install Portworx on Kubernetes, ensure that you're using a supported Kubernetes version:

| **Type** | **Supported Kubernetes Version** |
|---|---|
| On-prem Kubernetes | <ul><li>1.19</li><li>1.20</li><li>1.21</li><li>1.22</li><li>1.23</li></ul> |
| Managed Kubernetes | <ul><li>**KOPS:** 1.20</li><li>**GKE:** 1.19</li><li>**AKS:** 1.20</li><li>**EKS:** 1.20</li><li>**IKS:** 1.19</li><li>**PKS:** 1.17</li></ul> |
| Distribution Kubernetes | <ul><li>**Openshift 4.6:** 1.19 (Openshift version verified up to 4.6.48) </li><li>**Openshift 4.7:** 1.20 (Openshift version verified up to 4.7.37)</li><li>**Openshift 4.8:** 1.21 (Openshift version verified up to 4.8.20) </li><li>**Openshift 4.9:** 1.22 (Openshift version verified up to 4.9.7)</li></ul> |
| vSphere with Tanzu (TKGs) | <ul><li>v1.19.14+vmware.1-tkg.1.8753786</li><li>v1.20.9+vmware.1-tkg.1.a4cee5b</li></ul> |
| Tanzu Kubernetes Grid Integrated (TKGI) | <ul><li>PKS Version: 1.8.4-build.3; k8s Version: 1.17.17</li><li>PKS Version: 1.9.6-build.7; k8s Version: 1.18.18</li><li>PKS Version: 1.10.7-build.6; k8s Version: 1.19.16</li><li>PKS Version: 1.11.2-build.2; k8s Version: 1.20.06</li><li>PKS Version: 1.12.1-build.10; k8s Version: 1.21.5</li><li>PKS Version: 1.13.0-build.24; k8s Version: 1.22.2</li></ul> | 

{{<info>}}
**K3s users:** You must use CSI integration to generate / use PVCs.
{{</info>}}

## Installation

Whether you're using {{< pxEnterprise >}} or Essentials, you can install Portworx on the cloud or on-premises. Proceed to one of the following sections for install instructions.

{{<homelist series="k8s-install">}}

## Post-installation

If you have an existing Portworx cluster, continue to below sections for using and managing Portworx.

{{<homelist series2="k8s-postinstall">}}
