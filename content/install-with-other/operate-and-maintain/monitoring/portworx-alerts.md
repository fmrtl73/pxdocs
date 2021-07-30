---
title: Portworx Alerts
keywords: portworx, alerts, alarms, warnings, notifications
description: How to monitor alerts with Portworx.
---

## Portworx Alerts

Portworx provides a way to monitor your cluster using alerts. It has a predefined set of alerts which are listed below. The alerts are broadly classified into the following types based on the Resource on which it is raised

1. Cluster
2. Nodes
3. Disks
4. Volumes
5. Pools

Each alert has a severity from one of the following levels:

1. INFO
2. WARNING
3. ALARM

### List of Alerts

| Name | ResourceType | Severity | Description | Metric |
| :--- | :--- | :--- | :--- | :--- |
| DriveOperationFailure | DRIVE | ALARM | Triggered when a driver operation such as add or replace fails. | px_alerts_driveoperationfailure |
| DriveOperationSuccess | DRIVE | NOTIFY | Triggered when a driver operation such as add or replace succeeds. | px_alerts_driveoperationsuccess |
| DriveStateChange | DRIVE | WARNING | Triggered when there is a change in the driver state viz. Free Disk space goes below the recommended level of 10%. | px_alerts_drivestatechange |
| DriveStateChangeClear | DRIVE | WARNING | Triggered when the drive’s state gets cleared. | px_alerts_drivestatechangeclear |
| CloudDriveCreateWarning | DRIVE | ALARM | Warning during cloud drive creation | px_alerts_clouddrivecreatewarning |
| VolumeOperationFailureAlarm | VOLUME | ALARM | Triggered when a volume operation fails. Volume operations could be resize, cloudsnap, etc. The alert message will give more info about the specific error case. | px_alerts_volumeoperationfailurealarm |
| VolumeOperationSuccess | VOLUME | NOTIFY | Triggered when a volume operation such as resize succeeds. | px_alerts_volumeoperationsuccess |
| VolumeStateChange | VOLUME | WARNING | Triggered when there is a change in the state of the volume. | px_alerts_volumestatechange |
| IOOperation | VOLUME | ALARM | Triggered when an IO operation such as Block Read/Block Write fails. | px_alerts_iooperation |
| VolumeOperationFailureWarn | VOLUME | WARNING | Triggered when a volume operation fails. Volume operations could be resize, cloudsnap, etc. The alert message will give more info about the specific error case. | px_alerts_volumeoperationfailurewarn |
| VolumeSpaceLow | VOLUME | ALARM | Triggered when the free space available in a volume goes below a threshold. | px_alerts_volumespacelow |
| ReplAddVersionMismatch | VOLUME | WARNING | Triggered when a volume HA update fails with version mismatch. | px_alerts_repladdversionmismatch |
| CloudsnapOperationUpdate | VOLUME | NOTIFY | Triggered if a cloudsnap schedule is changed successfully. | px_alerts_cloudsnapoperationupdate |
| CloudsnapOperationFailure | VOLUME | ALARM | Triggered when a cloudsnap operation fails. | px_alerts_cloudsnapoperationfailure |
| CloudsnapOperationSuccess | VOLUME | NOTIFY | Triggered when a cloudsnap operation succeeds. | px_alerts_cloudsnapoperationsuccess |
| VolumeCreateSuccess | VOLUME | NOTIFY | Triggered when a volume is successfully created. | px_alerts_volumecreatesuccess |
| VolumeCreateFailure | VOLUME | ALARM | Triggered when a volume creation fails. | px_alerts_volumecreatefailure |
| VolumeDeleteSuccess | VOLUME | NOTIFY | Triggered when a volume is successfully deleted. | px_alerts_volumedeletesuccess |
| VolumeDeleteFailure | VOLUME | ALARM | Triggered when a volume deletion fails. | px_alerts_volumedeletefailure |
| VolumeMountSuccess | VOLUME | NOTIFY | Triggered when a volume is successfully mounted at the requested path. | px_alerts_volumemountsuccess |
| VolumeMountFailure | VOLUME | ALARM | Triggered when a volume cannot be mounted at the requested path. | px_alerts_volumemountfailure |
| VolumeUnmountSuccess | VOLUME | NOTIFY | Triggered when a volume is successfully unmounted. | px_alerts_volumeunmountsuccess |
| VolumeUnmountFailure | VOLUME | ALARM | Triggered when a volume cannot be unmounted. The alert message provides more info about the specific error case. | px_alerts_volumeunmountfailure |
| VolumeHAUpdateSuccess | VOLUME | NOTIFY | Triggered when a volume’s replication factor (HA factor) is successfully updated. | px_alerts_volumehaupdatesuccess |
| VolumeHAUpdateFailure | VOLUME | ALARM | Triggered when an update to volume’s replication factor (HA factor) fails. | px_alerts_volumehaupdatefailure |
| SnapshotCreateSuccess | VOLUME | NOTIFY | Triggered when a volume is successfully created. | px_alerts_snapshotcreatesuccess |
| SnapshotCreateFailure | VOLUME | ALARM | Triggered when a volume snapshot creation fails. | px_alerts_snapshotcreatefailure |
| SnapshotRestoreSuccess | VOLUME | NOTIFY | Triggered when a snapshot is successfully restored on a volume. | px_alerts_snapshotrestoresuccess |
| SnapshotRestoreFailure | VOLUME | ALARM | Triggered when the operation of restoring a snapshot fails. | px_alerts_snapshotrestorefailure |
| SnapshotIntervalUpdateFailure | VOLUME | ALARM | Triggered when an update of the snapshot interval for a volume fails. | px_alerts_snapshotintervalupdatefailure |
| SnapshotIntervalUpdateSuccess | VOLUME | NOTIFY | Triggered when a snapshot interval of a volume is successfully updated. | px_alerts_snapshotintervalupdatesuccess |
| VolumeExtentDiffSlow | VOLUME | WARNING | Volume extent diff is taking too long. | px_alerts_volumeextentdiffslow |
| VolumeExtentDiffOk | VOLUME | WARNING | Volume extent diff is okay. | px_alerts_volumeextentdiffok |
| SnapshotDeleteSuccess | VOLUME | NOTIFY | Triggered when a snapshot is successfully deleted. | px_alerts_snapshotdeletesuccess |
| SnapshotDeleteFailure | VOLUME | ALARM | Triggered when a snapshot delete is successfully deleted. | px_alerts_snapshotdeletefailure |
| VolumeSpaceLowCleared | VOLUME | NOTIFY | Triggered when the free disk space goes above the recommended level of 10%. | px_alerts_volumespacelowcleared |
| CloudMigrationUpdate | VOLUME | NOTIFY | Triggered if a cloud migration is updated. | px_alerts_cloudmigrationupdate |
| CloudMigrationSuccess | VOLUME | NOTIFY | Triggered when a cloud migration operation succeeds. | px_alerts_cloudmigrationsuccess |
| CloudMigrationFailure | VOLUME | ALARM | Triggered when a cloud migration operation fails. | px_alerts_cloudmigrationfailure |
| CloudsnapOperationWarning | VOLUME | WARNING | Triggered when a cloud snap operation encounters a problem. | px_alerts_cloudsnapoperationwarning |
| IOOperationWarning | VOLUME | WARNING | Io operation warning | px_alerts_iooperationwarning |
| FilesystemCheckSuccess | VOLUME | NOTIFY | Filesystem-Check fixed filesystem errors in volume | px_alerts_filesystemchecksuccess |
| FilesystemCheckFailed | VOLUME | WARNING | Filesystem-Check failed to fix errors in volume | px_alerts_filesystemcheckfailed |
| FilesystemCheckFoundErrors | VOLUME | WARNING | Filesystem-Check found errors in the filesystem | px_alerts_filesystemcheckfounderrors |
| VolumeResizeSuccessful | VOLUME | NOTIFY | Volume resize operation successful | px_alerts_volumeresizesuccessful |
| VolumeResizeDeferred | VOLUME | NOTIFY | Volume resize operation deferred to next mount | px_alerts_volumeresizedeferred |
| VolumeResizeFailed | VOLUME | ALARM | Volume resize operation failed | px_alerts_volumeresizefailed |
| VolumeAttachFailure | VOLUME | ALARM | Triggered when a volume cannot be attached to the requested node. | px_alerts_volumeattachfailure |
| VolumeDetachFailure | VOLUME | ALARM | Triggered when a volume cannot be detached from a node. | px_alerts_volumedetachfailure |
| VolumeRemoteDetach | VOLUME | NOTIFY | Triggered when a volume is detached from a remote node to allow attaching. | px_alerts_volumeremotedetach |
| VolumeCreateWarning | VOLUME | WARNING | Triggered when a volume creation encounters a non-critical problem. | px_alerts_volumecreatewarning |
| CloudMigrationCanceled | VOLUME | NOTIFY | Triggered when a cloud migration operation is canceled. | px_alerts_cloudmigrationcanceled |
| SharedV4FailoverNotAvailable | VOLUME | WARNING | Triggered when service failover is not available for a sharedv4 volume because it is not HA. | px_alerts_sharedv4failovernotavailable |
| SharedV4FailoverAvailable | VOLUME | NOTIFY | Triggered when service failover is available for a sharedv4 volume. | px_alerts_sharedv4failoveravailable |
| VolGroupOperationFailure | CLUSTER | ALARM | Triggered when a volume group operation fails. | px_alerts_volgroupoperationfailure |
| VolGroupOperationSuccess | CLUSTER | NOTIFY | Triggered when a volume group operation succeeds. | px_alerts_volgroupoperationsuccess |
| VolGroupStateChange | CLUSTER | WARNING | Triggered when a volume group’s state changes. | px_alerts_volgroupstatechange |
| ContainerOperationFailure | CLUSTER | ALARM | Container operation failed | px_alerts_containeroperationfailure |
| ContainerOperationSuccess | CLUSTER | ALARM | Container operation succeeded | px_alerts_containeroperationsuccess |
| ContainerStateChange | CLUSTER | ALARM | Container state changed | px_alerts_containerstatechange |
| LicenseExpiring | CLUSTER | WARNING | Warning triggers 7 days before the installed Portworx Enterprise or Trial license will expire (e.g. “PX-Enterprise license will expire in 6 days, 12:00”). It will also keep triggering after the license has expired (e.g. “Trial license expired 4 days, 06:22 ago”). | px_alerts_licenseexpiring |
| ClusterPairSuccess | CLUSTER | NOTIFY | Triggered when a cluster pair operation succeeds. | px_alerts_clusterpairsuccess |
| ClusterPairFailure | CLUSTER | ALARM | Triggered when a cluster pair operation fails. | px_alerts_clusterpairfailure |
| ClusterDomainAdded | CLUSTER | NOTIFY | Triggered when a cluster domain is added. | px_alerts_clusterdomainadded |
| ClusterDomainRemoved | CLUSTER | NOTIFY | Triggered when a cluster domain is removed. | px_alerts_clusterdomainremoved |
| ClusterDomainActivated | CLUSTER | NOTIFY | Triggered when a cluster domain is activated. | px_alerts_clusterdomainactivated |
| ClusterDomainDeactivated | CLUSTER | NOTIFY | Triggered when a cluster domain is deactivated. | px_alerts_clusterdomaindeactivated |
| MeteringAgentWarning | CLUSTER | WARNING | Triggered when the metering agent encounters a non-critical problem. | px_alerts_meteringagentwarning |
| MeteringAgentCritical | CLUSTER | ALARM | Triggered when the metering agent encounters a critical problem. | px_alerts_meteringagentcritical |
| ClusterLicenseUpdated | CLUSTER | NOTIFY | Triggered when a license is updated for a cluster. | px_alerts_clusterlicenseupdated |
| LicenseExpired | CLUSTER | ALARM | Triggered when the cluster license expires. | px_alerts_licenseexpired |
| LicenseLeaseExpiring | CLUSTER | WARNING | Triggered when the license lease is about to expire since the last lease refresh failed. | px_alerts_licenseleaseexpiring |
| LicenseLeaseExpired | CLUSTER | ALARM | Triggered when the license lease has expired since the last lease refresh failed. | px_alerts_licenseleaseexpired |
| RebalanceJobFinished | CLUSTER | ALARM | Rebalance job finished execution | px_alerts_rebalancejobfinished |
| RebalanceJobStarted | CLUSTER | ALARM | Rebalance job started execution | px_alerts_rebalancejobstarted |
| RebalanceJobPaused | CLUSTER | ALARM | Rebalance job paused execution | px_alerts_rebalancejobpaused |
| RebalanceJobCancelled | CLUSTER | ALARM | Rebalance job cancelled | px_alerts_rebalancejobcancelled |
| NodeStartFailure | NODE | ALARM | Triggered when a node in the Portworx cluster fails to start. | px_alerts_nodestartfailure |
| NodeStartSuccess | NODE | NOTIFY | Triggered when a node in the Portworx cluster successfully initializes. | px_alerts_nodestartsuccess |
| NodeStateChange | NODE | ALARM | Node state changed (i.e. it went down, came online etc.) | px_alerts_nodestatechange |
| NodeJournalHighUsage | NODE | ALARM | Triggered when a node’s timestamp journal usage is not within limits. | px_alerts_nodejournalhighusage |
| PXInitFailure | NODE | ALARM | Triggered when Portworx fails to initialize on a node. | px_alerts_pxinitfailure |
| PXInitSuccess | NODE | NOTIFY | Triggered when Portworx successfully initializes on a node. | px_alerts_pxinitsuccess |
| PXStateChange | NODE | WARNING | Triggered when the Portworx daemon shuts down in error. | px_alerts_pxstatechange |
| StorageVolumeMountDegraded | NODE | ALARM | Triggered when Portworx storage enters degraded mode on a node. | px_alerts_storagevolumemountdegraded |
| ClusterManagerFailure | NODE | ALARM | Triggered when Cluster manager on a Portworx node fails to start. The alert message will give more info about the specific error case. | px_alerts_clustermanagerfailure |
| KernelDriverFailure | NODE | ALARM | Triggered when an incorrect Portworx kernel module is detected. Indicates that Portworx is started with an incorrect version of the kernel module. | px_alerts_kerneldriverfailure |
| NodeDecommissionSuccess | NODE | NOTIFY | Triggered when a node is successfully decommissioned from Portworx cluster. | px_alerts_nodedecommissionsuccess |
| NodeDecommissionFailure | NODE | ALARM | Triggered when a node could not be decommissioned from Portworx cluster. | px_alerts_nodedecommissionfailure |
| NodeDecommissionPending | NODE | WARNING | Triggered when a node decommission is kept in pending state as it has data which is not replicated on other nodes. | px_alerts_nodedecommissionpending |
| NodeInitFailure | NODE | ALARM | Triggered when Portworx fails to initialize on a node. | px_alerts_nodeinitfailure |
| NodeScanCompletion | NODE | NOTIFY | Triggered when node media scan completes without error. | px_alerts_nodescancompletion |
| CloudsnapScheduleFailure | NODE | ALARM | Triggered if a cloudsnap schedule fails to configure. | px_alerts_cloudsnapschedulefailure |
| NodeMarkedDown | NODE | WARNING | Triggered when a Portworx node marks another node down as it is unable to connect to it. | px_alerts_nodemarkeddown |
| PXReady | NODE | NOTIFY | Triggered when Portworx is ready on a node. | px_alerts_pxready |
| StorageFailure | NODE | ALARM | Triggered when the provided storage drives could not be mounted by Portworx. | px_alerts_storagefailure |
| ObjectstoreFailure | NODE | ALARM | Triggered when an object store error is detected. | px_alerts_objectstorefailure |
| ObjectstoreSuccess | NODE | NOTIFY | Triggered upon a successful object store operation. | px_alerts_objectstoresuccess |
| ObjectstoreStateChange | NODE | NOTIFY | Triggered in response to a state change. | px_alerts_objectstorestatechange |
| SharedV4SetupFailure | NODE | WARNING | Triggered when the creation of a sharedv4 volume fails. | px_alerts_sharedv4setupfailure |
| NodeTimestampFailure | NODE | ALARM | Node timestamp journal failure | px_alerts_nodetimestampfailure |
| NodeJournalFailure | NODE | ALARM | Node journal failure | px_alerts_nodejournalfailure |
| StoragePoolFailure | NODE | ALARM | Storage Pool handling encountered an issue | px_alerts_storagepoolfailure |
| NodeDriverFailure | NODE | ALARM | Node Kernel Driver Failure | px_alerts_nodedriverfailure |
| StoragelessToStorageNodeTransitionFailure | NODE | ALARM | Triggered when a node fails to transition from a storageless type to a storage type. | px_alerts_storagelesstostoragenodetransitionfailure |
| StoragelessToStorageNodeTransitionSuccess | NODE | NOTIFY | Triggered when a node transitions from a storageless type to a storage type successfully. | px_alerts_storagelesstostoragenodetransitionsuccess |
| SecretsAuthFailed | NODE | WARNING | Secrets setup has failed | px_alerts_secretsauthfailed |
| LicenseServerDown | NODE | WARNING | Triggered when a node is unable to reach the license server. | px_alerts_licenseserverdown |
| FloatingLicenseSetupError | NODE | ALARM | Triggered when a node fails to setup a floating license. | px_alerts_floatinglicensesetuperror |
| NFSServerUnhealthy | NODE | WARNING | Triggered when the NFS server on this node is unhealthy. | px_alerts_nfsserverunhealthy |
| FileSystemDependency | NODE | ALARM | Triggered during Portworx installation if there’s a filesystem dependency failure. | px_alerts_filesystemdependency |
| RebootRequired | NODE | ALARM | Triggered when a node requires a reboot. | px_alerts_rebootrequired |
| TempFileSystemInitialization | NODE | ALARM | Triggered during Portworx installation if a node fails to initialize a temporary filesystem. | px_alerts_tempfilesysteminitialization |
| UnsupportedKernel | NODE | ALARM | Triggered during a Portworx installation if the node contains a kernel that is not supported by Portworx. | px_alerts_unsupportedkernel |
| InvalidDevice | NODE | ALARM | Triggered during Portworx installation if an invalid device is provided to Portworx as a storage device. | px_alerts_invaliddevice |
| NfsDependencyInstallFailure | NODE | ALARM | Triggered during Portworx installation if Portworx cannot install the NFS service. | px_alerts_nfsdependencyinstallfailure |
| NfsDependencyNotEnabled | NODE | ALARM | Triggered during Portworx installation if Portworx cannot enable the NFS service. | px_alerts_nfsdependencynotenabled |
| LicenseCheckFailed | NODE | ALARM | Triggered if a node fails a license check. | px_alerts_licensecheckfailed |
| PortworxStoppedOnNode | NODE | WARNING | Triggered if Portworx is stopped on a node. | px_alerts_portworxstoppedonnode |
| KvdbConnectionFailed | NODE | ALARM | Triggered if Portworx fails to connect to the KVDB. | px_alerts_kvdbconnectionfailed |
| InternalKvdbSetupFailed | NODE | ALARM | Triggered if Portworx fails to setup Internal KVDB on a node. | px_alerts_internalkvdbsetupfailed |
| PortworxMonitorImagePullFailed | NODE | ALARM | Triggered if Portworx fails to pull Portworx images during installation. | px_alerts_portworxmonitorimagepullfailed |
| PortworxMonitorPrePostExecutionFailed | NODE | ALARM | Triggered if Portworx fails to execute pre or post installation tasks. | px_alerts_portworxmonitorprepostexecutionfailed |
| PortworxMonitorMountValidationFailed | NODE | ALARM | Triggered if Portworx fails to validate mounts provided to Portworx container during installation. | px_alerts_portworxmonitormountvalidationfailed |
| PortworxMonitorSchedulerInitializationFailed | NODE | ALARM | Triggered if Portworx fails to initialize connection with scheduler during installation. | px_alerts_portworxmonitorschedulerinitializationfailed |
| PortworxMonitorServiceControlsInitializationFailed | NODE | ALARM | Triggered if Portworx fails to initialize the service controls during installation. | px_alerts_portworxmonitorservicecontrolsinitializationfailed |
| PortworxMonitorInstallFailed | NODE | ALARM | Triggered if Portworx installation fails. | px_alerts_portworxmonitorinstallfailed |
| MissingInputArgument | NODE | ALARM | Triggered if there’s a missing input install argument. | px_alerts_missinginputargument |
| PortworxMonitorImagePullInProgress | NODE | NOTIFY | Triggered when Portworx is pulling and extracting images during installation or upgrade. | px_alerts_portworxmonitorimagepullinprogress |
| InvalidArgument | NODE | ALARM | Invalid input argument | px_alerts_invalidargument |
| PXHostDependencyFailure | NODE | ALARM | Host does not meet dependencies for applied px configuration | px_alerts_pxhostdependencyfailure |
| KvdbConnectionWarning | NODE | WARNING | kvdb endpoint is not accessible | px_alerts_kvdbconnectionwarning |
| CallHomeFailure | NODE | ALARM | Call home failure | px_alerts_callhomefailure |
| CloudsnapSettingWarning | NODE | WARNING | Cloudsnap setting Warning | px_alerts_cloudsnapsettingwarning |
| Sharedv4ServerHighLoadWarn | NODE | WARNING | SharedV4 server high load detected | px_alerts_sharedv4serverhighloadwarn |
| NodeAttachmentsCordoned | NODE | NOTIFY | Volume attachments are disabled on this node | px_alerts_nodeattachmentscordoned |
| NodeAttachmentsUncordoned | NODE | NOTIFY | Volume attachments are re-enabled on this node | px_alerts_nodeattachmentsuncordoned |
| DrainAttachmentsJobStarted | NODE | NOTIFY | DrainAttachments job started execution | px_alerts_drainattachmentsjobstarted |
| DrainAttachmentsJobFinished | NODE | NOTIFY | DrainAttachments job finished execution | px_alerts_drainattachmentsjobfinished |
| DrainAttachmentsJobCancelled | NODE | ALARM | DrainAttachments job cancelled | px_alerts_drainattachmentsjobcancelled |
| CloudDriveTransferJobStarted | NODE | NOTIFY | CloudDriveTransfer job started execution | px_alerts_clouddrivetransferjobstarted |
| CloudDriveTransferJobInProgress | NODE | NOTIFY | CloudDriveTransfer job in progress | px_alerts_clouddrivetransferjobinprogress |
| CloudDriveTransferJobFinished | NODE | NOTIFY | CloudDriveTransfer job finished execution | px_alerts_clouddrivetransferjobfinished |
| CloudDriveTransferJobCancelled | NODE | ALARM | CloudDriveTransfer job cancelled | px_alerts_clouddrivetransferjobcancelled |
| DrainAttachmentsOperationWarning | NODE | WARNING | DrainAttachments operation warning | px_alerts_drainattachmentsoperationwarning |
| NodeUnsecured | NODE | ALARM | Triggered when an unsecure Portworx node tries to join a secured cluster. Indicates that the Portworx node is not configured correctly with the appropriate authentication security settings. | px_alerts_nodeunsecured |
| DiagCollectJobStarted | NODE | NOTIFY | DiagCollect job started execution | px_alerts_diagcollectjobstarted |
| DiagCollectJobInProgress | NODE | NOTIFY | DiagCollect job in progress | px_alerts_diagcollectjobinprogress |
| DiagCollectJobFinished | NODE | NOTIFY | DiagCollect job finished execution | px_alerts_diagcollectjobfinished |
| DiagCollectJobCancelled | NODE | ALARM | DiagCollect job cancelled | px_alerts_diagcollectjobcancelled |
| DiagCollectJobFailed | NODE | ALARM | DiagCollect job failed | px_alerts_diagcollectjobfailed |
| PoolExpandInProgress | POOL | NOTIFY | Triggered when a pool expand operation starts. | px_alerts_poolexpandinprogress |
| PoolExpandSuccessful | POOL | NOTIFY | Triggered when a pool expand operation succeeds. | px_alerts_poolexpandsuccessful |
| PoolExpandFailed | POOL | ALARM | Triggered when a pool expand operation fails. | px_alerts_poolexpandfailed |
