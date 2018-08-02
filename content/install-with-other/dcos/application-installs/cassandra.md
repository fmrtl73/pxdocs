---
title: Cassandra
---

The Portworx-Cassandra service can be found in the DC/OS catalog:

![Cassandra-PX in DCOS Universe](https://docs.portworx.com/images/dcos-cassandra-px-universe.png)

#### Default Install {#default-install}

If you want to use the defaults, you can now run the dcos command to install the service

```text
dcos package install --yes portworx-cassandra
```

You can also click on the “Install” button on the WebUI next to the service and then click “Install Package”. The default install will create PX volumes of size 5GB with 1 replica.

#### Advanced Install and Volume Options {#advanced-install-and-volume-options}

If you want to modify the default, click on the “Install” button next to the package on the DCOS UI and then click on “Advanced Installation”

Here you have the option to change the service name, volume name, volume size, and provide any additional options for the Portworx volume. You can also configure other Cassandra related parameters on this page including the number of Cassandra nodes.

![Cassandra-PX install options](https://docs.portworx.com/images/dcos-cassandra-px-install-options.png)

Click on “Review and Install” and then “Install” to start the installation of the service.

#### Install Status {#install-status}

Once you have started the install you can go to the Services page to monitor the status of the installation.

![Cassandra-PX on services page](https://docs.portworx.com/images/dcos-cassandra-px-service.png)

If you click on the Cassandra-PX service you should be able to look at the status of the nodes being created. There will be one service for the scheduler and one service each for the Cassandra nodes.

![Cassandra-PX install started](https://docs.portworx.com/images/dcos-cassandra-px-started-install.png)

When the Scheduler service as well as the Cassandra nodes are in Running \(green\) status, you should be ready to start using the Cassandra cluster.

![Cassandra-PX install finished](https://docs.portworx.com/images/dcos-cassandra-px-finished-install.png)

If you check your Portworx cluster, you should see multiple volumes that were automatically created using the options provided during install, one for each node of the Cassandra cluster.

![Cassandra-PX volumes](https://docs.portworx.com/images/dcos-cassandra-px-volume-list.png)

If you run the “dcos service” command you should see the portworx-cassandra service in ACTIVE state with 3 running tasks, one for each cassandra node.

```text
dcos service
NAME                            HOST                    ACTIVE  TASKS  CPU    MEM    DISK  ID
portworx-cassandra           10.0.0.179                  True     3    1.5  12288.0  0.0   5c6438b2-1f63-4c23-b62a-ad0a7d354a91-0115
marathon                     10.0.4.21                   True     1    1.0   1024.0  0.0   01d86b9c-ca2c-4c3c-9d9f-d3a3ef3e3911-0001
metronome                    10.0.4.21                   True     0    0.0    0.0    0.0   01d86b9c-ca2c-4c3c-9d9f-d3a3ef3e3911-0000
```

### Hyperconvergence {#hyperconvergence}

Running your Cassandra task on the same host as its data provides the best performance. This is called hyperconvergence and it is supported by the DC/OS Cassandra framework when using Portworx. When each Cassandra task is first launched, they create the required PX volumes. These volumes are created with data local to the node where they are first launched.

### Failover {#failover}

On subsequent launches of the same pod, for example in the case of a failover, the framework queries Portworx to figure out where the data for the volume resides and uses this to decide where the pod should be launched.

If there are not enough system resources \(like CPU, memory\) on the nodes where the data resides, the pod will eventually be started on a node where the data isn’t local. This helps ensure that the service can be bought online even when resource utilization and tasks have moved around in the cluster.

If the volume was created with a replication factor greater than 1, then the framework can decide to start the task on any of the nodes where the data is local.

### Scaling {#scaling}

You do not need to create additional PX volumes manually to scale up your cluster. Just go to the Cassandra service page, click on the three dots on the top right corner of the page, select “nodes”, scroll down and increase the nodes parameter to the desired nodes.

Click on “Review and Run” and then “Run Service”. The service scheduler should restart with the updated node count and create more Cassandra nodes with newly created PX volumes. Please make sure you have enough resources and nodes available to scale up the number of nodes. You also need to make sure Portworx is installed on all the agents in the DCOS cluster.

### Portworx Framework for Cassandra

The source code for these services can be found here: [Portworx DCOS-Commons Frameworks](https://github.com/portworx/dcos-commons)

> **Note:**  
> This framework is only supported directly by Portworx. Please contact support@portworx.com directly for any support issues related with using this framework.

Please make sure you have installed [Portworx on DCOS](https://docs.portworx.com/scheduler/mesosphere-dcos/install.html) before proceeding further.

### See Also {#see-also}

For further reading on Cassandra:

* [Cassandra Docker](https://portworx.com/use-case/cassandra-docker-container/) How to run Cassandra in Docker containers
* [Run multiple Cassandra rings on the same hosts](https://portworx.com/run-multiple-cassandra-clusters-hosts/)
* [Cassandra stress test with Portworx](https://docs.portworx.com/applications/cassandra-px-perf-test.html)
* [Run Cassandra on DCOS](https://docs.portworx.com/scheduler/mesosphere-dcos/cassandra.html)
* [Snapshotting Cassandra Container Volumes for CI/CD using Mesosphere DC/OS](https://portworx.com/snapshotting-cassandra-container-volumes-ci-using-mesosphere-dcos/)

