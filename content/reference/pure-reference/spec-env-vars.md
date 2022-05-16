---
title: Pure Storage FlashArray and FlashBlade storageCluster environment variables 
linkTitle: FlashBlade and FlashArray env vars
description: 
keywords: FlashArray, FlashBlade
weight: 600
---

This section provides configuration details for setting FlashArray and FlashBlade environment variables in the [`StorageCluster` object](/reference/crd/storage-cluster/). All fields are objects under the `spec.env[]` list:

```
spec:
  env:
  - name: PURE_DEFAULT_ENABLE_FB_NFS_SNAPSHOT
    value: "true"
```

| Name | Value | Type | Default |
| --- | --- | --- | --- |
| `PURE_DEFAULT_ENABLE_FB_NFS_SNAPSHOT` | Determines whether the .snapshot directory is enabled on FlashBlade filesystems or not. | `enumerated string`: `true` or `false` | `"true"` | 
| `PURE_ISCSI_LOGIN_TIMEOUT`| Login timeout in seconds | `string` | `"20"` |
| `PURE_ISCSI_ALLOWED_CIDRS`| Use this in cases where the FlashArray has additional network interfaces not on your own subnet. Use commas as the separator, e.g. 10.0.0.0/24,10.1.0.0/16. An empty string means all enabled iSCSI interfaces will be connected to. | `string` | `""` (empty string) |

