---
# required metadata

title: Configure SLES Cluster for SQL Server Availability Group | Microsoft Docs
description: 
author: MikeRayMSFT 
ms.author: mikeray 
manager: jhubbard
ms.date: 02/14/2017
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 85180155-6726-4f42-ba57-200bf1e15f4d

# optional metadata
# keywords: ""
# ROBOTS: ""
# audience: ""
# ms.devlang: ""
# ms.reviewer: ""
# ms.suite: ""
# ms.tgt_pltfrm: ""
# ms.custom: ""

---

# Configure SLES Cluster for SQL Server Availability Group

This guide provides instructions to create a two-node cluster for SQL Server on SUSE Linux Enterprise Server (SLES) 12 SP2. The clustering layer is based on SUSE [High Availability Extension (HAE)](https://www.suse.com/products/highavailability) built on top of [Pacemaker](http://clusterlabs.org/). 

For more details on cluster configuration, resource agent options, management, best practices, and recommendations, see [SUSE Linux Enterprise High Availability Extension 12 SP2](https://www.suse.com/documentation/sle-ha-12/index.html).

>[!NOTE]
>At this point, SQL Server's integration with Pacemaker on Linux is not as coupled as with WSFC on Windows. SQL Server on Linux is not cluster aware. Pacemaker controls all of the orchestration of the cluster resources as if SQL Server were a standalone instance. Also, virtual network name is specific to WSFC, there is no equivalent of the same in Pacemaker. On Linux, Always On Availability Group Dynamic Management Views (DMVs) will return empty rows. You can still create a listener to use it for transparent reconnection after failover, but you will have to manually register the listener name in the  DNS server with the IP used to create the virtual IP resource (as explained below).


## Prerequisites

To complete the end-to-end scenario below you need two machines to deploy the two nodes cluster. The steps below outline how to configure these servers.

## Setup and configure the operating system on each cluster node 

The first step is to configure the operating system on the cluster nodes. For this walk through, use SLES 12 SP2 with a valid subscription for the HA add-on.

### Install and configure SQL Server on each cluster node

1. Install and setup SQL Server on both nodes. For detailed instructions see [Install SQL Server on Linux](sql-server-linux-setup.md).

1. Designate one node as primary and the other as secondary, for purposes of configuration. Use these terms throughout this guide.

[!INCLUDE [SLES-Create-SQL-Login](../includes/ss-linux-cluster-pacemaker-create-login.md)]

1. Make sure nodes that are going to be part of the cluster can communicate to each other.

   The following example shows `/etc/hosts` with additions for two nodes named SLES1 and SLES2.

   ```
   127.0.0.1   localhost
   10.128.18.128 SLES1
   10.128.16.77 SLES2
   ```

   All cluster nodes must be able to access each other via SSH. Tools like `hb_report` or `crm_report` (for troubleshooting) and Hawk's History Explorer require passwordless SSH access between the nodes, otherwise they can only collect data from the current node. In case you use a non-standard SSH port, use the -X option (see `man` page). For example, if your SSH port is 3479, invoke a `crm_report` with:

   ```bash
   crm_report -X "-p 3479" [...]
   ```

   For additional information, see the [SLES Administration Guide - Miscellaneous section](http://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha/book_sleha.html#sec.ha.troubleshooting.misc).


## Configure an Always On Availability Group

On Linux servers configure the availability group and then configure the cluster resources. To configure the availability group, see [Configure Always On Availability Group for SQL Server on Linux](sql-server-linux-availability-group-overview.md)

## Install and configure Pacemaker on each cluster node

1. Install the High Availability extension

   For reference, see [Installing SUSE Linux Enterprise Server and High Availability Extension](https://www.suse.com/documentation/sle-ha-12/singlehtml/install-quick/install-quick.html#sec.ha.inst.quick.installation)

1. Install SQL Server resource agent package on both nodes.

   ```bash
   sudo zypper install mssql-server-ha
   ```

## Set up the first node

   Refer to [SLES installation instructions](http://www.suse.com/documentation/sle-ha-12/singlehtml/install-quick/install-quick.html#sec.ha.inst.quick.setup.1st-node)

1. Log in as `root` to the physical or virtual machine you want to use as cluster node.
2. Start the bootstrap script by executing:
   ```bash
   ha-cluster-init
   ```

   If NTP has not been configured to start at boot time, a message appears. 

   If you decide to continue anyway, the script will automatically generate keys for SSH access and for the Csync2 synchronization tool and start the services needed for both. 

3. To configure the cluster communication layer (Corosync): 

   a. Enter a network address to bind to. By default, the script will propose the network address of eth0. Alternatively, enter a different network address, for example the address of bond0. 

   b. Enter a multicast address. The script proposes a random address that you can use as default. 

   c. Enter a multicast port. The script proposes 5405 as default. 

   d. To configure `SBD ()`, enter a persistent path to the partition of your block device that you want to use for SBD. The path must be consistent across all nodes in the cluster. 
   Finally, the script will start the Pacemaker service to bring the one-node cluster online and enable the Web management interface Hawk2. The URL to use for Hawk2 is displayed on the screen. 
4. For any details of the setup process, check `/var/log/sleha-bootstrap.log`. You now have a running one-node cluster. Check the cluster status with crm status:

   ```bash
   crm status
   ```
5. The bootstrap procedure creates a Linux user named hacluster with the password linux. Replace the default password with a secure one as soon as possible: 

   ```bash
   passwd hacluster
   ```

## Add nodes to the existing cluster

If you have a cluster running with one or more nodes, add more cluster nodes with the ha-cluster-join bootstrap script. The script only needs access to an existing cluster node and will complete the basic setup on the current machine automatically. Follow the steps below:

If you have configured the existing cluster nodes with the `YaST` cluster module, make sure the following prerequisites are fulfilled before you run `ha-cluster-join`:
- The root user on the existing nodes has SSH keys in place for passwordless login. 
- Csync2 is configured on the existing nodes. For details, refer to Configuring Csync2 with YaST. 

1. Log in as root to the physical or virtual machine supposed to join the cluster. 
2. Start the bootstrap script by executing: 

   ```bash
   ha-cluster-join
   ```

   If NTP has not been configured to start at boot time, a message appears. 
3. If you decide to continue anyway, you will be prompted for the IP address of an existing node. Enter the IP address. 
4. If you have not already configured a passwordless SSH access between both machines, you will also be prompted for the root password of the existing node. 
After logging in to the specified node, the script will copy the Corosync configuration, configure SSH and Csync2, and will bring the current machine online as new cluster node. Apart from that, it will start the service needed for Hawk. If you have configured shared storage with OCFS2, it will also automatically create the mountpoint directory for the OCFS2 file system. 
5. Repeat the steps above for all machines you want to add to the cluster. 

6. For details of the process, check `/var/log/ha-cluster-bootstrap.log`. 

1. Check the cluster status with `crm status`. If you have successfully added a second node, the output will be similar to the following:

   ```bash
   crm status
   
   2 nodes configured
   1 resource configured
   Online: [ SLES1 SLES2 ]
   Full list of resources:
    admin_addr     (ocf::heartbeat:IPaddr2):       Started SLES1
   ```

   >[!NOTE]
   >`admin_addr` is the virtual IP cluster resource which is configured during initial one-node cluster setup.

After adding all nodes, check if you need to adjust the no-quorum-policy in the global cluster options. This is especially important for two-node clusters. For more information, refer to Section 4.1.2, Option no-quorum-policy. 

## Configure the cluster resources for SQL Server

Refer to [SLES Administration Guid](https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha/book_sleha.html#cha.ha.manual_config)

### Create availability group resource

The following command creates and configures the availability group resource for 2 replicas of availability group [ag1]. Run the command on one of the nodes in the cluster:

```bash
crm configure
#primitive ag_cluster \
   ocf:mssql:ag \
   params ag_name="ag1" \
   op monitor interval="10" role="Master" \
   op monitor interval="11" role="Slave"
#ms ms-ag_cluster ag_cluster \
   meta master-max="1" master-node-max="1" clone-max="2" \
   clone-node-max="1"
commit
```

### Create virtual IP resource

If you did not create the virtual IP resource when you ran `ha-cluster-init` you can create this resource now. The following command creates a virtual IP resource. Run on one node.

```bash
crm configure
# primitive admin_addr \
   ocf:heartbeat:IPaddr2 \
   params ip=<10.9.9.180> \
      cidr_netmask=<24> \
   op monitor interval="12s"
```

### Add colocation constraint
To set colocation constraint for the virtual IP to run on same node as the master, run the following command on one node:

```bash
crm configure
colocation vip_on_master inf: \
    admin_addr ms-ag_cluster:Master
commit
```

### Add ordering constraint
The colocation constraint has an implicit ordering constraint. It moves the virtual IP resource before it moves the availability group resource. By default the sequence of events is: 
1. User issues resource migrate to the availability group master from node1 to node2.
2. The virtual IP resource stops on node 1.
3. The virtual IP resource starts on node 2. At this point, the IP address temporarily points to node 2 while node 2 is still a pre-failover secondary. 
4. The availability group master on node 1 is demoted to slave.
5. The availability group slave on node 2 is promoted to master. 

To prevent the IP address from temporarily pointing to the node with the pre-failover secondary, add an ordering constraint. 
To add an ordering constraint, run the following command on one node: 

```bash
crm configure
   order ag_first inf: ms-ag_cluster:promote admin_addr:start
commit
For more information, see Show Cluster Resources.
```

## Manual failover

>[!IMPORTANT]
>After you configure the cluster and add the availability group as a cluster resource, you cannot use Transact-SQL to fail over the availability group resources. SQL Server cluster resources on Linux are not coupled as tightly with the operating system as they are on a Windows Server Failover Cluster (WSFC). SQL Server is not aware of the presence of the cluster. All orchestration is done through the cluster management tools. In SLES use `crm`. 

>[!IMPORTANT]
>If the availability group is a cluster resource, there is a known issue in current release where manual failover to an asynchronous replica does not work. This will be fixed in the upcoming release. Manual or automatic failover to a synchronous replica will succeed. 

Manage failover of the availability group with `crm`. Do not initiate failover with Transact-SQL. To manually failover to cluster node2, run the following command. 

```bash
crm resource migrate ms-ag_cluster sles1
```

>[!NOTE]
>At this time manual failover to an asynchronous replica does not work properly. This will be fixed in a future release.

For additional details see:
- [Managing cluster resources](https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha/book_sleha.html#sec.ha.config.crm).   
- [HA Concepts](https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha/book_sleha.html#cha.ha.concepts)
- [Pacemaker Quick Reference](https://github.com/ClusterLabs/pacemaker/blob/master/doc/pcs-crmsh-quick-ref.md) 

## Removing Nodes From An Existing Cluster
If you have a cluster running (with at least two nodes), you can remove single nodes from the cluster with the `sleha-remove` bootstrap script. You need to know the IP address or host name of the node you want to remove from the cluster. Follow the steps below:

1. Log in as root to one of the cluster nodes. 
2. Start the bootstrap script by executing: 

   ```branch
   ha-cluster-remove -c IP_ADDR_OR_HOSTNAME
   ```

   The script enables the `sshd`, stops the pacemaker service on the specified node, and propagates the files to synchronize with `Csync2` across the remaining nodes.

   If you specified a host name and the node to remove cannot be contacted (or the host name cannot be resolved), the script will inform you and ask whether to remove the node anyway. If you specified an IP address and the node cannot be contacted, you will be asked to enter the host name and to confirm whether to remove the node anyway. 

3. To remove more nodes, repeat the step above. 
4. For details of the process, check `/var/log/ha-cluster-bootstrap.log`.

## Removing the High Availability Extension Software From a Machine
To remove the High Availability Extension software from a machine that you no longer need as cluster node, proceed as follows:

1. Stop the cluster service:

   ```bash
   rcopenais stop
   ```

2. Remove the High Availability Extension add-on:

   ```
   zypper rm -t products sle-hae
   ```

## Next steps

[Create SQL Server Availability Group](sql-server-linux-availability-group-configure.md)
