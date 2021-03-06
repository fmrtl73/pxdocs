---
title: CouchDB
description: Installing DC/OS with CouchDB service.
keywords: Portworx, Mesosphere, DCOS, CouchDB
---

DC/OS provides a CouchDB service that makes it easy to deploy and manage CouchDB on Mesosphere DC/OS. This guide will help you to install and run the containerized CouchDB service backed by Portworx volumes for [persistent DCOSstorage](https://portworx.com/use-case/persistent-storage-dcos/).

The source code for these services can be found here: [Portworx DCOS-Commons Frameworks](https://github.com/portworx/dcos-commons)

{{<info>}}
**Note:**  
This framework is only supported directly by Portworx. Please contact support@portworx.com directly for any support issues related with using this framework.
{{</info>}}

Please make sure you have installed [Portworx on DCOS](/install-with-other/dcos) before proceeding further.

### Install {#install}

#### Adding the repository for the service {#adding-the-repository-for-the-service}

For this step you will need to login to a node which has the dcos cli installed and is authenticated to your DCOS cluster.

Run the following command to add the repository to your DCOS cluster:

```text

dcos package repo add --index=0 portworx-couchdb-aws https://universe-converter.mesosphere.com/transform?url=https://px-dcos-dev.s3.amazonaws.com/autodelete7d/portworx-couchdb/20180108-191814-r2aJF3ibkK2ltugj/stub-universe-portworx-couchdb.json
```

Once you have run the above command you should see the `portworx-couchdb` service available in your universe

![Couchdb-PX in DCOS Universe](/img/dcos-portworx-couchdb-universe.png)

#### Default Install {#default-install}

If you want to use the defaults, you can now run the dcos command to install the service

```text
dcos package install --yes portworx-couchdb
```

You can also click on the “Install” button on the WebUI next to the service and then click “Install Package”. The default install will create PX volumes of size 10GB with 1 replica.

#### Advanced Install and Volume Options {#advanced-install-and-volume-options}

If you want to modify the default, click on the “Install” button next to the package on the DCOS UI and then click on “Advanced Installation”

Here you have the option to change the service name, volume name, volume size, and provide any additional options for the Portworx volume. You can also configure other CouchDB related parameters on this page including the number of CouchDB nodes.

![portworx-couchdb install options](/img/dcos-portworx-couchdb-install-options.png)

Click on “Review and Install” and then “Install” to start the installation of the service.

#### Install Status {#install-status}

Once you have started the install you can go to the Services page to monitor the status of the installation.

![portworx-couchdb on services page](/img/dcos-portworx-couchdb-service.png)

If you click on the `portworx-couchdb` service you should be able to look at the status of the nodes being created. There will be one service for the scheduler and one service each for the CouchDB nodes.

![portworx-couchdb install started](/img/dcos-portworx-couchdb-started-install.png)

When the Scheduler service as well as the CouchDB nodes are in Running \(green\) status, you should be ready to start using the CouchDB cluster.

![portworx-couchdb install finished](/img/dcos-portworx-couchdb-finished-install.png)

If you check your Portworx cluster, you should see multiple volumes that were automatically created using the options provided during install, one for each node of the CouchDB cluster.

![portworx-couchdb volumes](/img/dcos-portworx-couchdb-volume-list.png)

If you run the “dcos service” command you should see the `portworx-couchdb` service in ACTIVE state with 3 running tasks, one for each CouchDB node.

```text
dcos service
NAME                   HOST      ACTIVE  TASKS  CPU   MEM     DISK   ID
marathon          192.168.65.90   True     2    1.5  2048.0   0.0    b69b8ce2-fe89-4688-850c-9a70438fc8f3-0000
metronome         192.168.65.90   True     0    0.0   0.0     0.0    b69b8ce2-fe89-4688-850c-9a70438fc8f3-0001
portworx             a1.dcos      True     2    0.6  2048.0  1024.0  b69b8ce2-fe89-4688-850c-9a70438fc8f3-0022
portworx-couchdb     a1.dcos      True     3    1.5  6144.0   0.0    b69b8ce2-fe89-4688-850c-9a70438fc8f3-0029
```

### Verify Setup {#verify-setup}

From the DCOS client, install the new command for `portworx-couchdb`

```text
dcos package install portworx-couchdb --cli
```

Find out all CouchDB client endpoints

```text
dcos portworx-couchdb endpoints cluster-port
{
  "address": [
    "192.168.65.131:5984",
    "192.168.65.121:5984",
    "192.168.65.111:5984"
  ],
  "dns": [
    "couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984",
    "couchdb-1-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984",
    "couchdb-2-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984"
  ]
}
```

Connect to the master node to access the CouchDB service

```text
dcos node ssh --master-proxy --leader
```

From the DCOS master node, run the CouchDB REST API to any of the nodes on port `5984`. The default credentials are `admin:password` for accessing the REST APIs. A json output of the members in the CouchDB cluster from one of the nodes is shown below.

```text
curl -s http://admin:password@couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984/_membership | python -m json.tool
{
    "all_nodes": [
        "couchdb@couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory",
        "couchdb@couchdb-1-install.portworx-couchdb.autoip.dcos.thisdcos.directory",
        "couchdb@couchdb-2-install.portworx-couchdb.autoip.dcos.thisdcos.directory"
    ],
    "cluster_nodes": [
        "couchdb@couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory",
        "couchdb@couchdb-1-install.portworx-couchdb.autoip.dcos.thisdcos.directory",
        "couchdb@couchdb-2-install.portworx-couchdb.autoip.dcos.thisdcos.directory"
    ]
}
```

To verify the CouchDB cluster we try to create a new database `testdb` and add a simple document to it.

```text
curl -s -X PUT http://admin:password@couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984/testdb -d {} | python -m json.tool
{
    "ok": true
}

curl -s -X PUT http://admin:password@couchdb-0-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984/testdb/001 -d '{"name":"Alice"}' | python -m json.tool
{
    "id": "001",
    "ok": true,
    "rev": "1-4cb726bf80cfbb1457e6cce338834b1f"
}
```

Verify the inserted document

```text
curl -s http://admin:password@couchdb-1-install.portworx-couchdb.autoip.dcos.thisdcos.directory:5984/testdb/001 | python -m json.tool
{
    "_id": "001",
    "_rev": "1-4cb726bf80cfbb1457e6cce338834b1f",
    "name": "Alice"
}
```

### Scaling {#scaling}

You do not need to create additional PX volumes manually to scale up your cluster. Just go to the CouchDB service page, click on the three dots on the top right corner of the page, select “nodes”, scroll down and increase the nodes parameter to the desired nodes.

Click on “Review and Run” and then “Run Service”. The service scheduler should restart with the updated node count and create more CouchDB nodes with newly created PX volumes. Please make sure you have enough resources and nodes available to scale up the number of nodes. You also need to make sure Portworx is installed on all the agents in the DCOS cluster.

You can also increase the capacity of your CouchDB data nodes by using the `pxctl volume update` command without taking the service offline.
