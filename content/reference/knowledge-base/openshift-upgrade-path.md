---
title: Operator OpenShift upgrade path
keywords: Operator, OpenShift, OCP, upgrade
description: Review which OpenShift versions you should upgrade to if you're running Portworx using the Operator.
weight: 6
series: kb
---

## Portworx 2.8.0

This upgrade path was tested with Portworx 2.8.0 and Operator 1.6.0.

{{<info>}}
**NOTE:** If you are not on the same OCP version as in the tested upgrade example shown below, please upgrade to the next available version that is used in the upgrade path and continue as shown in the table below.
{{</info>}}

<table>
<tr>
<td> <strong>OCP channel</strong> </td><td> stable-4.5 </td><td> stable-4.6 </td><td> stable-4.7 </td>
</tr>
<tr>
<td> <strong>OCP version</strong> </td><td>4.5.41</td><td>4.6.46</td><td>4.7.33</td>
</tr>
<tr>
<td> <strong>Kubernetes version</strong> </td><td>v1.18.3+d8ef5ad</td><td>v1.19.0+d5ed12c</td><td>v1.20.0+bbbc079</td>
</tr>
</table>
