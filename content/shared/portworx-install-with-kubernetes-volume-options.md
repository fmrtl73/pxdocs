---
title: Volume options
keywords: portworx, volumes, pvc
description: Volume options
hidden: true
---

| Name              	| Description                                                                                                                                                                                                                                                            	| Example                	|
|-------------------	|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|------------------------	|
| fs                	| Specifies a filesystem to be laid out: xfs\|ext4                                                                                                                                                                                                                               	| fs: "ext4"               	|
| repl              	| Specifies the replication factor for the volume: 1\|2\|3                                                                                                                                                                                                                                	| repl: "3"                	|
| sharedv4            	| Creates a globally shared namespace volume which can be used by multiple pods over NFS with POSIX compliant semantics                                                                                                                                                                                   	| sharedv4: "true"         	|
| sharedv4_svc_type | Indicates the mechanism Kubernetes will use for locating your `sharedv4` volume. If you use this flag and there's a failover of the nodes running your `sharedv4` volume, you no longer need to restart your pods. Possible values are: ClusterIP or LoadBalancer. | sharedv4_svc_type: "ClusterIP" |
| sharedv4_failover_strategy		| Specifies how aggressively to fail over to a new server for a Sharedv4 or Sharedv4 Service volume (Valid Values: aggressive, normal)	| sharedv4_failover_strategy: "aggressive"
| io_priority       	| Specifies IO Priority: low\|medium\|high. The default is low | io_priority: "high" |
| io_profile       	| Overrides I/O algorithm that Portworx uses for a volume. For more information about IO profiles, see the [IO profiles](/concepts/io-profiles) section of the documentation.                                                                                                                                                                                                                                      	| io_profile: "db"    	|
| group             	| Specifies the group a volume should belong too. Portworx restricts replication sets of volumes of the same group on different nodes. If the force group option 'fg' is set to true, the volume group rule is strictly enforced. By default, it's not strictly enforced. 	| group: "volgroup1"       	|
| fg                	| Enforces volume group policy. If a volume belonging to a group cannot find nodes for its replication sets which don't have other volumes of the same group, the volume creation will fail.                                                                    	| fg: "true"             	|
| label             	| Arbitrary key=value labels that can be applied on a volume                                                                                                                                                                                             	| label: "name=mypxvol"    	|
| nodes             	| Specifies comma-separated Portworx Node IDs to use for replication sets of the volume                                                                                                                                                                                           	| nodes: "minion1,minion2" 	|
| ephemeral             | Creates the ephemeral volumes | ephemeral: false |
| size		            | Specifies a volume size in GB (default 1) | size: "1073741824" |
| scale		            | Auto-scales the volume to a maximum number. (Valid Range: [1 1024]) (default 1) | scale: 1 |
| block_size		    | Specifies a block size in Bytes (default 4096)     | block_size: "4096" |
| queue_depth		    | Specifies a block device queue depth. (Valid Range: [1 256]) (default 128)        | queue_depth: 128 |
| snap_interval		    | Specifies an interval in minutes at which periodic snapshots will be triggered. Set to 0 to disable snapshots | 
| snap_schedule		    | Specifies the name of the snapshot schedule policy created using the `pxctl sched-policy` command |Refer to [this page](/reference/cli/storagepolicy/) for examples |
| secret_key		    | Specifies Secret Key to be used for encrypting volumes | Refer to the [Secrets Management](/key-management/) page for examples. |
| zones		            | Specify comma-separated zone names in which the volume replicas should be distributed |
| racks		            | Specify comma-separated rack names in which the volume replicas should be distributed |
| async_io	            | Enables asynchronous IO for backing up storage    | async_io: false |
| csi_mount_options		| Specifies the mounting options for a volume through CSI |
| sharedv4_mount_options| Specifies a comma-separated list of Sharedv4 NFS client mount options provided as key=value pairs |
| proxy_endpoint		| Specifies the endpoint address of the external NFS share Portworx is proxying | proxy_endpoint: "nfs://\<nfs-share-endpoint>" |
| proxy_nfs_subpath		| Specifies the sub-path from the NFS share to which this proxy volume has access to| 
| proxy_nfs_exportpath	| Exports path for NFS proxy volume | proxy_nfs_exportpath: "/\<mount-path>" |
| export_options		| Defines the export options. Currently, only NFS export options are supported for Sharedv4 volumes |
| mount_options		    | Specifies the mounting options for a volume when it is attached and mounted |
| best_effort_location_provisioning	| Requested nodes, zones, racks are optional |
| direct_io		        | Enables Direct IO on a volume | direct_io: "true" |
| scan_policy_trigger	| Specifies the trigger point on which filesystem check is triggered. Valid Values: none, on_mount, on_next_mount |
| scan_policy_action		| Specifies a filesystem scan action to be taken when triggered. Valid Values: none, scan_only, scan_repair |
| force_unsupported_fs_type		| Forces a filesystem type that is not supported. The driver may still refuse to use the type | force_unsupported_fs_type: false |
| match_src_vol_provision		| Provisions the restore volume on the same pools as the source volume (src volume must exist) |
| nodiscard		           | Mounts the volume with nodiscard option. This is useful when the volume undergoes a large amount of block discards and later the application rewrites to these discarded block making the discard work done by Portworx useless. This option must be used along with `auto_fstrim`. | nodiscard: false |
| auto_fstrim		    | Enables `auto_fstrim` on a volume and requires the `nodiscard` option to be set. Refer to [this page](/reference/cli/volume-trim/) for more details.|	auto_fstrim: true |
| storagepolicy		    | Creates a volume on the Portworx cluster that follows the specified set of specs/rules. Refer [this page](/reference/cli/storagepolicy/) for more details. | 
| backend		| Specifies which storage backend Portworx is going to provide direct access to. (Valid Values: pure_block, pure_file)	 |backend:  "pure_block" |
| pure_export_rules		| Specifies the export rules for exporting a Pure Flashblade volume |	 pure_export_rules: "*(rw)" |
| io_throttle_rd_iops	| Specifies maximum Read IOPs a volume will be throttled to. Refer to [this page](/portworx-install-with-kubernetes/storage-operations/io-throttling/) for more details. |   io_throttle_rd_iops: "1024" |
| io_throttle_wr_iops	| Specifies maximum Write IOPs a volume will be throttled to. Refer to [this page](/portworx-install-with-kubernetes/storage-operations/io-throttling/) for more details.	| io_throttle_wr_iops: "1024" |
| io_throttle_rd_bw		| Specifies maximum Read bandwidth a volume will be throttled to. Refer to [this page](/portworx-install-with-kubernetes/storage-operations/io-throttling/) for more details.	| io_throttle_rd_bw: "10" |
| io_throttle_wr_bw		| Specifies maximum Write bandwidth a volume will be throttled to. Refer to [this page](/portworx-install-with-kubernetes/storage-operations/io-throttling/) for more details.	|  io_throttle_wr_bw: "10" |
| aggregation_level 	| Specifies the number of replication sets the volume can be aggregated from                                                                                                                                                                                             	| aggregation_level: "2"   	|
| sticky     	| Creates sticky volumes that cannot be deleted until the flag is disabled                                                                                                                                                                                                      	| sticky: "true"     	|
| journal     	| Indicates if you want to use journal device for the volume's data. This will use the journal device that you used when installing Portworx. This is useful to absorb frequent syncs from short bursty workloads. Default: false                                                                                                                                                                                             	| journal: "true"     	|
| secure | Creates an encrypted volume. For details about how you can create encrypted volumes, see the [Create encrypted PVCs](/portworx-install-with-kubernetes/storage-operations/create-pvcs/create-encrypted-pvcs/) page.| secure: "true" |
| placement_strategy | Flag to refer the name of the <code>VolumePlacementStrategy</code>. For example:<br></br><code>apiVersion: storage.k8s.io/v1</code><br><code>kind: StorageClass</code><br><code>metadata:</code><br><code>&nbsp;name: postgres-storage-class</code><br><code>provisioner: kubernetes.io/portworx-volume</code><br><code>parameters:</code><br><code>&nbsp;placement_strategy: "postgres-volume-affinity"</code><br><br> For details about how to create and use `VolumePlacementStrategy`, see [this page] (/portworx-install-with-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/create-use-volplacestrat/).| placement_strategy: "postgres-volume-affinity" |
| snapshotschedule.<br>stork.<br>libopenstorage.<br>org | Creates scheduled snapshots with Stork. For example:<br></br><code>snapshotschedule.stork.libopenstorage.org/default-schedule:</code><br> <code>&nbsp;schedulePolicyName: daily</code><br> <code>&nbsp;annotations:</code><br><code>&nbsp;&nbsp;portworx/snapshot-type: local</code><br><code>snapshotschedule.stork.libopenstorage.org/weekly-schedule:</code><br><code>&nbsp;schedulePolicyName: weekly</code><br><code>&nbsp;annotations:</code><br><code>&nbsp;&nbsp;portworx/snapshot-type: cloud</code><br><code>&nbsp;&nbsp;portworx/cloud-cred-id: <credential-uuid></code><br></br>{{<info>}}
<b>Note:</b> This example references two schedules:<br><ul><li>The <code>default-schedule</code> backs up volumes to the local Portworx cluster daily.</li><li>The <code>weekly-schedule</code> backs up volumes to cloud storage every week.</li>
{{</info>}}For details about how you can create scheduled snapshots with Stork, see the [Scheduled snapshots](/portworx-install-with-kubernetes/storage-operations/create-snapshots/scheduled/) page. | 
