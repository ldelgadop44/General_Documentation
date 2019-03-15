# ETCD Installation
This guide show the way to install ETCD cluster.

## Definition
Etcd is an open-source distributed key value store that provides shared configuration and service discovery for Container Linux clusters. etcd runs on each machine in a cluster and gracefully handles leader election during network partitions and the loss of the current leader.

The next picture explain graphically the ETCD configuration in the cluster:

![Image of ETCD_Cluster](../images/ETCD_Cluster.png)

Like we see, have 3 virtual machines, each of one have assigned ip address, with correct navigation and DNS configuration.

## Addressing
* **VM1:** 192.168.1.223
* **VM2:** 192.168.1.224
* **VM3:** 192.168.1.227

## Prerequisites

**1.** Create 3 virtual Machines on virtualbox.

**2.** Install CoreOs in each of them. See the guide: [CoreOS installation](https://github.com/ldelgadop44/General_Documentation/blob/coreos_documentation/CoreOS_Installation.md).

**3.** Verify Internet connection and DNS configuration.

## Installation

#### Virtual Machine 1

**1.** Execute: ```sudo mkdir -p /etc/systemd/system/etcd-member.service.d/ && sudo vim /etc/systemd/system/etcd-member.service.d/20-clct-etcd-member.conf```

Inside file *20-clct-etcd-member.conf* put the next lines
```
[Service]
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_IMAGE_TAG=v3.2.15"
Environment="ETCD_OPTS=--name infra0 \
--initial-advertise-peer-urls http://192.168.1.224:2380 \
--listen-peer-urls http://192.168.1.224:2380 \
--listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
--advertise-client-urls http://192.168.1.224:2379 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster infra0=http://192.168.1.224:2380,infra1=http://192.168.1.223:2380,infra2=http://192.168.1.227:2380 \
--initial-cluster-state new \
--advertise-client-urls http://192.168.1.224:2379"
```

**2.** The, execute: 
* ```systemctl enable etcd-member``` -> To enable the service when the machine starts
* ```rm -rf /var/lib/etcd``` -> To delete existing data into this directory
* ```sudo systemctl daemon-reload``` -> For apply changes in the service files
* ```systemctl start etcd-member``` -> For start etcd daemon

#### Virtual Machine 2

**1.** Execute: ```sudo mkdir -p /etc/systemd/system/etcd-member.service.d/ && sudo vim /etc/systemd/system/etcd-member.service.d/20-clct-etcd-member.conf```

Inside file *20-clct-etcd-member.conf* put the next lines
```
[Service]
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_IMAGE_TAG=v3.2.15"
Environment="ETCD_OPTS=--name infra1 \
--initial-advertise-peer-urls http://192.168.1.223:2380 \
--listen-peer-urls http://192.168.1.223:2380 \
--listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
--advertise-client-urls http://192.168.1.223:2379 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster infra0=http://192.168.1.224:2380,infra1=http://192.168.1.223:2380,infra2=http://192.168.1.227:2380 \
--initial-cluster-state new \
--advertise-client-urls http://192.168.1.223:2379"
```

**2.** The, execute: 
* ```systemctl enable etcd-member```
* ```rm -rf /var/lib/etcd```
* ```sudo systemctl daemon-reload```
* ```systemctl start etcd-member```

#### Virtual Machine 3

**1.** Execute: ```sudo mkdir -p /etc/systemd/system/etcd-member.service.d/ && sudo vim /etc/systemd/system/etcd-member.service.d/20-clct-etcd-member.conf```

Inside file *20-clct-etcd-member.conf* put the next lines
```
[Service]
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_IMAGE_TAG=v3.2.15"
Environment="ETCD_OPTS=--name infra2 \
--initial-advertise-peer-urls http://192.168.1.227:2380 \
--listen-peer-urls http://192.168.1.227:2380 \
--listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
--advertise-client-urls http://192.168.1.227:2379 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster infra0=http://192.168.1.224:2380,infra1=http://192.168.1.223:2380,infra2=http://192.168.1.227:2380 \
--initial-cluster-state new \
--advertise-client-urls http://192.168.1.227:2379"
```

**2.** The, execute: 
* ```systemctl enable etcd-member```
* ```rm -rf /var/lib/etcd```
* ```sudo systemctl daemon-reload```
* ```systemctl start etcd-member```

## Test application

* With this command list nodes of the cluster ```etcdctl member list```, and you get the next exit
```
20d77eebb2106ded: name=infra0 peerURLs=http://192.168.1.224:2380 clientURLs=http://192.168.1.224:2379 isLeader=false
914f6f494cd55b75: name=infra1 peerURLs=http://192.168.1.223:2380 clientURLs=http://192.168.1.223:2379 isLeader=false
9b04564d1787cc1c: name=infra2 peerURLs=http://192.168.1.227:2380 clientURLs=http://192.168.1.227:2379 isLeader=true
```

* Then, we can create a test key on VM1; for this execute ```etcdctl set key1 prueba```

* And then, we can consult the same key on other nodes of cluster (VM2, VM3) ```etcdctl get key1```

* Finally, we can poweroff any virtual machine and see cluster behavior. with the command ```etcdctl member list``` or ```etcdctl cluster-health```