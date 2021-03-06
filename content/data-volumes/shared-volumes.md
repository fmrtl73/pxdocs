---
title: Shared Volumes
description: Portworx Shared Volumes work with NFS and give the ability for multiple containers to access the same volumes.  Watch the video to see how!
keywords: portworx, PX-Developer, container, Shared Volume, storage
weight: 4
---

Through shared volumes \(also known as a **global namespace**\), a single volume’s filesystem is concurrently available to multiple containers running on multiple hosts.

{{<info>}}
**Note:**  
You do not need to use shared volumes to have your data accessible on any host in the cluster. Any PX volumes can be exclusively accessed from any host as long as they are not simultaneously accessed. Shared volumes are for providing simultaneous \(concurrent or shared\) access to a volume from multiple hosts at the same time.
{{</info>}}

A typical pattern is for a single container to have one or more volumes. Conversely, many scenarios would benefit from multiple containers being able to access the same volume, possibly from different hosts. Accordingly, the shared volume feature enables a single volume to be read/write accessible by multiple containers. Example use cases include:

* A technical computing workload sourcing its input and writing its output to a shared volume.
* Scaling a number of Wordpress containers based on load while managing a single shared volume.
* Collecting logs to a central location

{{<info>}}
**Note:**  
Usage of shared volumes for databases is not recommended, since they have a small metadata overhead.
{{</info>}}
