---
title: Hadoop and HDFS
---

This guide will help you to install the Hadoop service on your DCOS cluster backed by PX volumes for persistent storage. It will create 3 Journal Nodes, 2 Name Nodes, 2 Nodes for the Zookeeper Failover Controller, 3 Data Nodes and 3 Yarn Nodes. The Data and Yarn nodes will be co-located on the same physical host.

The number of Data and Yarn nodes can be set during install. They can also be updated after install to scale the service.

The source code for these services can be found here: [Portworx DCOS-Commons Frameworks](https://github.com/portworx/dcos-commons)

> **Note:**  
> This framework is only supported directly by Portworx. Please contact support@portworx.com directly for any support issues related with using this framework.

Please make sure you have installed [Portworx on DCOS](https://docs.portworx.com/scheduler/mesosphere-dcos/install.html) before proceeding further.

The Portworx-hadoop service can be found in the DC/OS catalog:

![Hadoop-PX in DCOS Universe](https://docs.portworx.com/images/dcos-hadoop-px-universe.png)

### Installation {#installation}

#### Default Install {#default-install}

If you want to use the defaults, you can now run the dcos command to install the service

```text
dcos package install --yes portworx-hadoop
```

You can also click on the “Install” button on the WebUI next to the service and then click “Install Package”.

#### Advanced Install {#advanced-install}

If you want to modify the default, click on the “Install” button next to the package on the DCOS UI and then click on “Advanced Installation”

Here you have the option to change the service name, volume name, volume size, and provide any additional options that you want to pass to the docker volume driver. You can also configure other Hadoop related parameters on this page including the number of Data and Yarn nodes for the Hadoop clsuter.

![Hadoop-PX install options](https://docs.portworx.com/images/dcos-hadoop-px-install-options.png)

Click on “Review and Install” and then “Install” to start the installation of the service.

### Install Status {#install-status}

Once you have started the install you can go to the Services page to monitor the status of the installation.

![Hadoop-PX on services page](https://docs.portworx.com/images/dcos-hadoop-px-service.png)

If you click on the Hadoop-PX service you should be able to look at the status of the nodes being created. There will be one service for the scheduler and one each for the Journal, Name, Zookeeper, Data and Yarn nodes.

![Hadoop-PX install started](https://docs.portworx.com/images/dcos-hadoop-px-started-install.png)

When the Scheduler service as well as all the Hadoop containers nodes are in Running \(green\) status, you should be ready to start using the Hadoop cluster.

![Hadoop-PX install finished](https://docs.portworx.com/images/dcos-hadoop-px-finished-install.png)

If you check your Portworx cluster, you should see multiple volumes that were automatically created using the options provided during install, one for each of the Journal, Name and Data nodes.

![Hadoop-PX volumes](https://docs.portworx.com/images/dcos-hadoop-px-volume-list.png)

If you run the “dcos service” command you should see the portworx-hadoop service in ACTIVE state with 13 running tasks

```text
dcos service
NAME                         HOST                    ACTIVE  TASKS  CPU    MEM    DISK  ID                                         
portworx-hadoop           10.0.0.135                  True     13   9.0  32768.0  0.0   5c6438b2-1f63-4c23-b62a-ad0a7d354a91-0113  
marathon                  10.0.4.21                   True     1    1.0   1024.0  0.0   01d86b9c-ca2c-4c3c-9d9f-d3a3ef3e3911-0001  
metronome                 10.0.4.21                   True     0    0.0    0.0    0.0   01d86b9c-ca2c-4c3c-9d9f-d3a3ef3e3911-0000  
```

###  Running Hadoop with Portworx on DCOS {#running-hadoop-with-portworx-on-dcos}

Here is a short video that shows how to install and run Hadoop on Portworx with DCOS  
 

### Scaling the Data Nodes {#scaling-the-data-nodes}

You do not need to create additional volumes of perform to scale up your cluster. Just go to the Hadoop service page, click on the three dots on the top right corner of the page, select “Data”, scroll down and increase the nodes parameter to the desired nodes.

Click on “Review and Run” and then “Run Service”. The service scheduler should restart with the updated node count and create more Data nodes. Please make sure you have enough resources and nodes available to scale up the number of nodes. You also need to make sure Portworx is installed on all the agents in the DCOS cluster.

You can also increase the capacity of your HDFS Data nodes by using the `pxctl volume update` command without taking the service offline

