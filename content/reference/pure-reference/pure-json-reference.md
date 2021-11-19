---
title: Pure Storage FlashArray and FlashBlade JSON file reference
linkTitle: FlashBlade and FlashArray JSON
description: 
keywords: FlashArray, FlashBlade
weight: 4
---

Portworx uses a single secret for both FlashArray and FlashBlade configuration. If you're planning on using both together, specify all of your FlashBlade and FlashArray entries in a single JSON file.  

```text
{
    "FlashArrays": [
        {
            "MgmtEndPoint": "<first-fa-management-endpoint>",
            "APIToken": "<first-fa-api-token>"
        },
        {
            "MgmtEndPoint": "<second-fa-management-endpoint>",
            "APIToken": "<second-fa-api-token>"
        }
    ],
    "FlashBlades": [
        {
            "MgmtEndPoint": "<fb-management-endpoint>",
            "APIToken": "<fb-api-token>",
            "NFSEndPoint": "<fb-nfs-endpoint>"
        },
        {
            "MgmtEndPoint": "<fb-management-endpoint>",
            "APIToken": "<fb-api-token>",
            "NFSEndPoint": "<fb-nfs-endpoint>"
        }
    ]
}
```

### FlashArray object reference

|**Key**|**Value**| **Required?** |
|----|----|----|
| `MgmtEndPoint` | Your FlashArray management endpoint.<br/>**Data type:** string | Yes |
| `APIToken` | Your FlashArray API token.<br/>**Data type:** string | Yes |

### FlashBlade object reference

|**Key**|**Value**| **Required?** |
|----|----|----|
| `MgmtEndPoint` | Your FlashBlade management endpoint.<br/>**Data type:** string | Yes |
| `APIToken` | Your FlashBlade API token.<br/>**Data type:** string | Yes |
| `NFSEndPoint` | Your FlashBlade NFS endpoint.<br/>**Data type:** string |  Yes |

## Related topics

* [Disk provisioning on Pure Storage FlashArray](/cloud-references/auto-disk-provisioning/pure-flash-array/)
* [Configure FlashBlade as a Direct Access filesystem](/portworx-install-with-kubernetes/storage-operations/create-pvcs/pure-flashblade/)