---
title: Etcd Quick Setup
keywords: etcd, quick setup, install, maintenance, kvdb
hidden: true
disableprevnext: true
aliases:
  - /reference/knowledge-base/etcd-quick-setup/
  - /reference/etcd-quick-setup/
---

Following guide will setup a 3 node etcd cluster. Etcd will be running as a systemd services on the nodes.

### Prerequisites

Ensure that you meet the following requirements for setup:

* You must use version 3 of the `etcd` API.

* The three nodes that form the etcd cluster must meet the following requirements:

    * The nodes should have static IPs.
    * The nodes should have systemd installed.

### Download and install etcd

1. Get the `etcd` tar ball from the official CoreOS site:

    ```text
    ETCD_VER=v3.2.7 && curl -L "https://storage.googleapis.com/etcd/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz" -o /tmp/etcd.tar.gz
    ```

    You can replace the value of __ETCD_VER__ with the `etcd` version that you wish to install, but you must be using API version 3.

2. Untar the `etcd` tar ball:

    ```text
    rm -rf /tmp/etcd && mkdir -p /tmp/etcd
    tar xzvf /tmp/etcd.tar.gz -C /tmp/etcd --strip-components=1
    ```

3. Install the `etcd` binaries:

    ```text
    sudo cp /tmp/etcd/etcd /usr/local/bin/
    sudo cp /tmp/etcd/etcdctl /usr/local/bin/
    ```

4. Repeat the preceding steps on all the 3 nodes before moving forward.

### Set up systemd unit files

1. Create a `systemd` environment file `/etc/etcd.conf` which has the IPs of all the nodes:

    ```text
    cat /etc/etcd.conf
    ```

    ```output
    # SELF_IP is the IP of the node where this file resides.
    SELF_IP=70.0.40.154
    # IP of Node 1
    NODE_1_IP=70.0.40.153
    # IP of Node 2
    NODE_2_IP=70.0.40.154
    # IP of Node 3
    NODE_3_IP=70.0.40.155
    ```

    You can copy the above contents into your `/etc/etcd.conf` and replace the IPs with the IPs of your three nodes.

2. Create a copy of the above file on all three of the nodes. The contents of the file will remain same except for the __SELF_IP__, which will correspond to the node's IP where the file resides.

3. Create a `systemd` unit file for the etcd3 service:

    ```text
    cat /etc/systemd/system/etcd3.service
    ```

    ```output
    [Unit]
    Description=etcd
    Documentation=https://github.com/coreos/etcd
    Conflicts=etcd.service
    Conflicts=etcd2.service

    [Service]
    Type=notify
    Restart=always
    RestartSec=25s
    LimitNOFILE=40000
    TimeoutStartSec=20s
    EnvironmentFile=/etc/etcd.conf
    ExecStart=/bin/sh -c "/usr/local/bin/etcd --name etcd-${SELF_IP} --data-dir /var/lib/etcd --quota-backend-bytes 8589934592 --auto-compaction-retention 3 --listen-client-urls http://${SELF_IP}:2379,http://localhost:2379 --advertise-client-urls http://${SELF_IP}:2379,http://localhost:2379 --listen-peer-urls http://${SELF_IP}:2380 --initial-advertise-peer-urls http://${SELF_IP}:2380 --initial-cluster 'etcd-${NODE_1_IP}=http://${NODE_1_IP}:2380,etcd-${NODE_2_IP}=http://${NODE_2_IP}:2380,etcd-${NODE_3_IP}=http://${NODE_3_IP}:2380' --initial-cluster-token my-etcd-token --initial-cluster-state new"

    [Install]
    WantedBy=multi-user.target
    ```

    You can copy the above contents into your `/etc/systemd/system/etcd3.service`. This configuration file has all the `etcd` configuration parameters set to the recommended values. No further changes are required in this file as it reads the IPs from the custom EnvironmentFile `/etc/etcd.conf`

4. Create a copy of the above file on all the 3 nodes. The contents of the file will remain same on all the nodes.

### Start etcd3 systemd service.

Once the `systemd` files are setup correctly on all the 3 nodes, run the following commands on all 3 nodes to start `etcd`.

```text
sudo systemctl daemon-reload
sudo systemctl enable etcd3
sudo systemctl start etcd3
```

### Validate etcd setup

Run the following commands to check if etcd is setup correctly, replacing `<etd-node-1>`, `<etd-node-2>`, and `<etd-node-3>` with the hostnames or IPs of your etcd nodes:

```text
etcdctl -w table --endpoints=<etcd-node-1>:2379,<etcd-node-2>:2379,<etcd-node-3>:2379 endpoint status
etcdctl --endpoints=<etcd-node-1>:2379,<etcd-node-2>:2379,<etcd-node-3>:2379 endpoint health
```

```output
member 56a14e6f53fae617 is healthy: got healthy result from http://70.0.40.154:2379
member 7e34afa2930c40e5 is healthy: got healthy result from http://70.0.40.155:2379
member 8ff90a5cbffc52d4 is healthy: got healthy result from http://70.0.40.153:2379
cluster is healthy
```

Make sure the IPs are displayed correctly and the cluster health is reported as `healthy`.
