How to get going with fabric8 v1 in Fuse 6.1 (part1)
====================================================
Author: Matt Robson

Technologies: Fuse, Fabric8

Product: Fuse 6.1

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
* Java 1.7
* JBoss Fuse 6.1

Prerequisites
-------------
* 3-5 servers (or VMs) for a 3 or 5 node ensemble
* Red Hat Enterprise Linux (or any other supported varient)

The Build Out                                                                                                             
-------------
Create the root container:

	mkdir /opt/fuse/

	mv jboss-fuse-full-6.1.0.redhat-379.zip /opt/fuse/

	unzip jboss-fuse-full-6.1.0.redhat-379.zip

	cd jboss-fuse-6.1.0.redhat-379/etc/

Edit:

	vi users.properties

Uncomment:

	admin=admin,admin

Edit:

	vi org.apache.karaf.management.cfg

Uncomment:

	jmxRole=admin

Edit:

	vi org.apache.karaf.shell.cfg

Uncomment:

	sshRole=admin

	cd /opt/fuse/jboss-fuse-6.1.0.redhat-379/bin

Start the server:

	./start

Start a client session:

	./client

Create the first Fabric node:

	JBossFuse:admin@root> fabric:create --wait-for-provisioning --verbose --clean --new-user admin --new-user-role admin --new-user-password admin --zookeeper-password passwd --resolver manualip --manual-ip fusefabric1.lab.com
	Waiting for container: root
	Waiting for container root to provision.
	Using specified zookeeper password:passwd

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

	17:00:04,179 | INFO  | Thread-49        | ZooKeeper                        | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Initiating client connection, connectString=fusefabric1.lab.com:2181 sessionTimeout=60000 watcher=org.apache.curator.ConnectionState@3e679c6e
	17:00:04,924 | INFO  | .com:2181) | ClientCnxn                       | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Opening socket connection to server fusefabric1.lab.com/10.10.183.203:2181. Will not attempt to authenticate using SASL (unknown error)
	17:00:05,085 | INFO  | .com:2181) | ClientCnxn                       | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Socket connection established to fusefabric1.lab.com/10.10.183.203:2181, initiating session
	17:00:05,292 | INFO  | .com:2181) | ClientCnxn                       | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | Session establishment complete on server fusefabric1.lab.com/10.10.183.203:2181, sessionid = 0x14d25d7c6500002, negotiated timeout = 40000
	17:00:05,297 | INFO  | d-49-EventThread | ConnectionStateManager           | 53 - io.fabric8.fabric-zookeeper - 1.0.0.redhat-379 | State change: CONNECTED

If you do a container-list from the root node, you can see root2 has been added but is shutdown and not yet provisioned.

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       false       fabric

Once you see it fully shutdown, you can start it back up again:

	17:00:12,685 | INFO  | FelixStartLevel  | Activator                        | 4 - org.ops4j.pax.logging.pax-logging-api - 1.7.2 | Disabling SLF4J API support.
	17:00:12,685 | INFO  | FelixStartLevel  | Activator                        | 4 - org.ops4j.pax.logging.pax-logging-api - 1.7.2 | Disabling Jakarta Commons Logging API support.
	17:00:12,685 | INFO  | FelixStartLevel  | Activator                        | 4 - org.ops4j.pax.logging.pax-logging-api - 1.7.2 | Disabling Log4J API support.
	17:00:12,685 | INFO  | FelixStartLevel  | Activator                        | 4 - org.ops4j.pax.logging.pax-logging-api - 1.7.2 | Disabling Avalon Logger API support.
	17:00:12,685 | INFO  | FelixStartLevel  | Activator                        | 4 - org.ops4j.pax.logging.pax-logging-api - 1.7.2 | Disabling JULI Logger API support.

Start the server:

	./start

From the root container, you can see it transition through the stages of provisioning:

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             downloading
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             resolving
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             installing
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             finalizing
	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             success

You will also notice that a new log file has been created, karaf.log.  Once the server is running and has joined the Fabric, all related logging can now be found here.

Repeat steps 11-12 to join your remaining containers.

Once that is finished, you will be left with a 1 node ensemble and 5 nodes in your fabric.

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                         [provision status]
	root*                          1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success
	root2                          1.0       true        fabric                                             success
	root3                          1.0       true        fabric                                             success
	root4                          1.0       true        fabric                                             success
	root5                          1.0       true        fabric                                             success

To finish, we want to add our new root containers to the ensemble.

	JBossFuse:admin@root> fabric:ensemble-add root2 root3 root4 root5
	This will change of the zookeeper connection string.
	Are you sure want to proceed(yes/no):yes
	Updated Zookeeper connection string: fusefabric1.lab.com:2182,fusefabric2.lab.com:2181,fusefabric3.lab.com:2181,fusefabric4.lab.com:2181,fusefabric5.lab.com:2181

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
	root4                          1.0       true        fabric, fabric-ensemble-0001-4                     success
	root5                          1.0       true        fabric, fabric-ensemble-0001-5                     success

Done and Done.

To expand on your fabric and add some functionality, continue on to part 2 <https://github.com/mrobson/fuse-fabric8-ssh-containers>.
