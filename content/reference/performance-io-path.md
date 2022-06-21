---
title: Portworx performance optimization options in the IO path
description: Describes various performance optimizations Portworx has in the IO path.
hidden: true
---

This document describes various performance optimizations Portworx has in the IO path.

## Data integrity in the IO path

Portworx ensures data integrity in the entire IO path. Specifically, it protects against memory corruption, corruption over the network, and bit rot on disk. Where possible, Portworx will seamlessly repair invaild blocks. 

### Data integrity on write
When data is written from the application to the block device, the blocks are checksummed. The data along with the checksums is transferred end to end on the IO path. It is revalidated to protect against memory or network corruption and before writing to stable media.

### Data integrity and repair on read

In the read path, the data is read along with checksums from disk. If the checksum doesn't match, the block is fetched from a replica and re-written. The checksum is computed again before writing the response to the client to protect against memory or network corruption.

## In-memory checksum

Portworx maintains a per block checksum of all data blocks coming into and going out of the system. For write requests, checksums are computed as soon as data arrives at the storage process. For read requests, checksums are computed as soon as data is read from the local pool. For remote reads, both data and checksums are sent over the wire to the remote node. Checksum verification takes place before the data is written to stable media or returned to the application. 

This feature is on by default and protects against in-memory corruption as well as corruption over the network.
  
## On-disk checksum and repair on read

Portworx provides a feature to detect media errors such as bit rot and bad blocks. Portworx uses a copy-on-write filesystem on the backing storage pool which stores the virtual volumes. All writes arriving at the pool are checksummed and are stored separately from data blocks.

In the read path, the data is read along with checksums. If the checksum doesn't match, it is treated as an IO error from the pool, and the recovery mechanism is triggered. This reads the block from a good replica when possible, and writes it to the mismatched replica. The checksum is computed again before writing the response to the client to protect against memory or network corruption.
 
While the space overhead is very small (around 0.1%), there is a performance overhead of this feature. The checksum needs copy-on-write to be enabled, which has the following implications:

* All writes are treated as new writes. The filesystem has to take the entire block allocation path, which can be avoided for in-place overwrites.
* All blocks directly or indirectly involved in the write get modified (direct data block, indirect data blocks, metadata blocks). The extra modification causes high write amplification and fragmentation.
* Additional background work is required to maintain block reference counting because each write goes through an `alloc_new` and `free_old` cycle. 

Although the overhead depends on the workload (small block vs. large block, random vs. sequential, sync vs. async), we expect around 20-30% overhead on a general purpose use case.

This feature is off by default. It can be enabled by using the `–cow_ondemand=off` flag at volume creation or in the StorageClass. This feature also gets enabled automatically if snapshots are enabled.

{{<info>}}
**NOTE:** As mentioned, this feature typically helps with media errors only, such as bit rot and bad blocks. 
{{</info>}}

## Sync workloads

Portworx implements synchronous replication for HA, meaning all writes are synchronously mirrored across all replicas. The replica placement is failure domain aware. Replicas are placed across failure domains (for example, Rack, DC, AZ) when possible.

At runtime, Portworx uses this location awareness to optimize database workloads. Most database workloads consist of small synchronous write IOs. This type of workload causes the backend to force a flush operation to make the write stable. The flush operation requires the filesystem to serialize operations by adding barriers, then wait for data and metadata to be stable on disk. When possible, Portworx mitigates this cost by combining back to back flush operations. It acknowledges the write after making sure that the write has made it to the page cache (kernel) on all replicas. Each replica has its internal timers and threshold which trigger a flush operation. The maximum timer threshold is capped at 100ms.

In case of failure across multiple fault domains (a kernel panic across all fault domains within a 100ms window), the pool state rolls back to at most 100ms in the past.

This feature is enabled by default. It can be disabled by using the `--io_profile=sequential` volume update flag.

## Related topics

* [IO profiles](/concepts/io-profiles/)
* [Performance tuning](/install-with-other/operate-and-maintain/performance-and-tuning/tuning/)
