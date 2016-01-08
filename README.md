How to get going with fabric8 v1 in Fuse 6.2.1 (part1)
======================================================
Author: Matt Robson

Technologies: Fuse, Fabric8

Product: Fuse 6.2.1

Breakdown                                                                                                                     
---------                                                                                                                     
This is a command based example to demonstrate how to build out a Fabric Ensemble using join.  We will also outline some tuning options available to make your Fabric more robust.

For more information see:

* <https://access.redhat.com/site/documentation/JBoss_Fuse/> for more information about using Red Hat JBoss Fuse
* <http://www.jboss.org/products/fuse/overview/> for more information about the upstream community                                              
* <http://fabric8.io/> for more information about fabric8

System Requirements
-------------------
Before building out your Fabric, you will need:
* Java 1.7 or 1.8
* JBoss Fuse 6.2.1

Prerequisites
-------------
* 3-5 servers (or VMs) for a 3 or 5 node ensemble
* Red Hat Enterprise Linux (or any other supported varient)

The Build Out                                                                                                             
-------------
Create the root container:

	mkdir /opt/fuse/

	mv jboss-fuse-full-6.2.1.redhat-084.zip /opt/fuse/

	unzip jboss-fuse-full-6.2.1.redhat-084.zip

	cd jboss-fuse-6.2.1.redhat-084/etc/

Edit:

	vi users.properties

Uncomment:

	admin=admin,admin,manager,viewer,Monitor, Operator, Maintainer, Deployer, Auditor, Administrator, SuperUser

	cd /opt/fuse/jboss-fuse-6.2.1.redhat-084/bin

Start the server:

	./start

Start a client session:

	./client

Create the first Fabric node:

	JBossFuse:admin@root> fabric:create --wait-for-provisioning --verbose --clean --new-user admin --new-user-role admin --new-user-password admin --zookeeper-password passwd --resolver manualip --manual-ip fusefabric1.lab.com
	Waiting for container: root
	Waiting for container root to provision.

Verify it was created:

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success

Repeat steps 1-10 on 2 or 4 more server so you can create a 3 or 5 node ensemble. Once your other servers are ready, proceed to the next step.

Join nodes 2-5 to the fabric:

	JBossFuse:admin@root> fabric:join --zookeeper-password passwd --resolver manualip --manual-ip fusefabric2.lab.com fusefabric1.lab.com:2181 root2
	You are about to change the container name. This action will restart the container.
	The local shell will automatically restart, but ssh connections will be terminated.
	The container will automatically join: fusefabric1.lab.com:2181 the cluster after it restarts.
	Do you wish to proceed (yes/no):yes

In the logs, you will see it connect to your root node and then shutdown.

	16:35:24,866 | INFO  | Thread-55        | ZooKeeper                        | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | Initiating client connection, connectString=fusefabric1.lab.com:2181 sessionTimeout=60000 watcher=org.apache.curator.ConnectionState@29a1ffa
	16:35:24,906 | INFO  | redhat.com:2181) | ClientCnxn                       | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | Opening socket connection to server fusefabric1.lab..com/10.10.183.203:2181
	16:35:24,912 | INFO  | redhat.com:2181) | ClientCnxn                       | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | Socket connection established to fusefabric1.lab.com/10.10.183.203:2181, initiating session
	16:35:24,925 | INFO  | redhat.com:2181) | ClientCnxn                       | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | Session establishment complete on server fusefabric1.lab.com/10.10.183.203:2181, sessionid = 0x152232706670002, negotiated timeout = 40000
	16:35:24,935 | INFO  | d-55-EventThread | ConnectionStateManager           | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | State change: CONNECTED
	16:35:26,026 | INFO  | Thread-55        | ZooKeeper                        | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | Session: 0x152232706670002 closed
	16:35:26,026 | INFO  | d-55-EventThread | ClientCnxn                       | 145 - io.fabric8.fabric-zookeeper - 1.2.0.redhat-621084 | EventThread shut down

You will see the container shutdown and then start back up again:

	16:35:26,095 | INFO  | FelixShutdown    | BlueprintExtender                | 23 - org.apache.aries.blueprint.core - 1.4.4 | Destroying BlueprintContainer for bundle io.hawt.hawtio-json-schema-mbean/1.4.0.redhat-621084
	16:35:26,100 | INFO  | FelixShutdown    | BlueprintExtender                | 23 - org.apache.aries.blueprint.core - 1.4.4 | Destroying BlueprintContainer for bundle org.apache.camel.karaf.camel-karaf-commands/2.15.1.redhat-621084
	16:35:26,232 | INFO  | FelixShutdown    | BlueprintExtender                | 23 - org.apache.aries.blueprint.core - 1.4.4 | Destroying BlueprintContainer for bundle activemq-karaf/5.11.0.redhat-621084
	........
	2016-01-08 16:35:55,056 | INFO  | 2.0.1-1-thread-1 | DeploymentAgent                  | 142 - io.fabric8.fabric-agent - 1.2.0.redhat-621084 | Validating baseline information
	2016-01-08 16:35:55,084 | INFO  | 2.0.1-1-thread-1 | patch-management                 | 1 - io.fabric8.patch.patch-management - 1.2.0.redhat-621084 | Configuring patch management system
	2016-01-08 16:36:00,640 | INFO  | 2.0.1-1-thread-1 | patch-management                 | 1 - io.fabric8.patch.patch-management - 1.2.0.redhat-621084 | No user changes detected
	2016-01-08 16:36:03,091 | INFO  | 2.0.1-1-thread-1 | Agent                            | 142 - io.fabric8.fabric-agent - 1.2.0.redhat-621084 | Done.

From the root container, you can see it transition through the stages of provisioning:

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             analyzing
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             downloading (1 pending)
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             installing
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             validating baseline information
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             success

Repeat steps 11-12 to join your remaining containers.

Once that is finished, you will be left with a 1 node ensemble and 5 nodes in your fabric.

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             success
	root3                          1.0       true        fabric                                             success

To finish, we want to add our new root containers to the ensemble.

	JBossFuse:admin@root> fabric:ensemble-add root2 root3
	This will change of the zookeeper connection string.
	Are you sure want to proceed(yes/no):yes
	Updated Zookeeper connection string: fusefabric1.lab.com:2182,fusefabric2.lab.com:2181,fusefabric3.lab.com:2181

Logs:

	2015-05-06 11:07:58,618 | INFO  | Thread-128284    | CuratorFrameworkImpl             | mework.imps.CuratorFrameworkImpl  222 | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Starting
	2015-05-06 11:07:58,620 | INFO  | Thread-128284    | ZooKeeper                        | org.apache.zookeeper.ZooKeeper    438 | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Initiating client connection, connectString=fusefabric1.lab.com:2182,fusefabric2.lab.com:2181,fusefabric3.lab.com:2181,fusefabric4.lab.com:2181,fusefabric5.lab.com:2181 sessionTimeout=30000 watcher=org.apache.curator.ConnectionState@777146b2
	2015-05-06 11:07:58,669 | INFO  | admin-1-thread-1 | FabricConfigAdminBridge          | figadmin.FabricConfigAdminBridge  173 | 67 - io.fabric8.fabric-configadmin - 1.0.0.redhat-379 | Updating configuration io.fabric8.zookeeper.server.c838e8e0-9fe0-49aa-bd19-87e7eb182c93
	2015-05-06 11:07:58,673 | INFO  | 19-87e7eb182c93) | ZooKeeperServerFactory           | bootstrap.ZooKeeperServerFactory  101 | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Creating zookeeper server with: {server.3=fusefabric3.lab.com:2888:3888, component.name=io.fabric8.zookeeper.server, server.2=fusefabric2.lab.com:2888:3888, server.1=fusefabric1.lab.com:2888:3888, server.id=1, initLimit=10, syncLimit=5, service.factoryPid=io.fabric8.zookeeper.server, fabric.zookeeper.pid=io.fabric8.zookeeper.server-0001, clientPort=2182, clientPortAddress=0.0.0.0, service.pid=io.fabric8.zookeeper.server.c838e8e0-9fe0-49aa-bd19-87e7eb182c93, tickTime=2000, component.id=60, dataDir=data/zookeeper/0001, server.5=fusefabric5.lab.com:2888:3888, server.4=fusefabric4.lab.com:2888:3888}
	2015-05-06 11:08:08,230 | INFO  | agent-2-thread-1 | DeploymentAgent                  | io.fabric8.agent.DeploymentAgent  753 | 60 - io.fabric8.fabric-agent - 1.0.0.redhat-379 | Done.

Once it's completed, you will now see all your nodes as part of the ensemble:

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, jboss-fuse-full, fabric-ensemble-0001-1    success
	root2                          1.0       true        fabric, fabric-ensemble-0001-2                     success
	root3                          1.0       true        fabric, fabric-ensemble-0001-3                     success

Done and Done.

To expand on your fabric and add some functionality, continue on to part 2 <https://github.com/mrobson/fuse-fabric8-ssh-containers>.
