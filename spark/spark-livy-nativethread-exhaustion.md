#### Issue : Livy Server was down in HDInsight Spark Production Cluster [(Spark 2.1 on Linux (HDI 3.6)]
#### Attempting to restart results in the following error stack.
##### Found the following entries in the latest livy Logs 
~~~~
 17/07/27 17:52:50 INFO CuratorFrameworkImpl: Starting
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:zookeeper.version=3.4.6-29--1, built on 05/15/2017 17:55 GMT
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:host.name=10.0.0.66
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.version=1.8.0_131
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.vendor=Oracle Corporation
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.class.path= <DELETED>
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.library.path= <DELETED>
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.io.tmpdir=/tmp
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.compiler=<NA>
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.name=Linux
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.arch=amd64
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.version=4.4.0-81-generic
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.name=livy
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.home=/home/livy
 17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.dir=/home/livy
 17/07/27 17:52:50 INFO ZooKeeper: Initiating client connection, connectString=zk2-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk3-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk6-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181 sessionTimeout=60000 watcher=org.apache.curator.ConnectionState@25fb8912
 17/07/27 17:52:50 INFO StateStore$: Using ZooKeeperStateStore for recovery.
 17/07/27 17:52:50 INFO ClientCnxn: Opening socket connection to server 10.0.0.61/10.0.0.61:2181. Will not attempt to authenticate using SASL (unknown error)
 17/07/27 17:52:50 INFO ClientCnxn: Socket connection established to 10.0.0.61/10.0.0.61:2181, initiating session
 17/07/27 17:52:50 INFO ClientCnxn: Session establishment complete on server 10.0.0.61/10.0.0.61:2181, sessionid = 0x25d666f311d00b3, negotiated timeout = 60000
 17/07/27 17:52:50 INFO ConnectionStateManager: State change: CONNECTED
 17/07/27 17:52:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
 17/07/27 17:52:50 INFO AHSProxy: Connecting to Application History server at headnodehost/10.0.0.67:10200
 Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
  at java.lang.Thread.start0(Native Method)
  at java.lang.Thread.start(Thread.java:717)
  at com.cloudera.livy.Utils$.startDaemonThread(Utils.scala:98)
  at com.cloudera.livy.utils.SparkYarnApp.<init>(SparkYarnApp.scala:232)
  at com.cloudera.livy.utils.SparkApp$.create(SparkApp.scala:93)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:117)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:116)
  at com.cloudera.livy.server.batch.BatchSession.<init>(BatchSession.scala:137)
  at com.cloudera.livy.server.batch.BatchSession$.recover(BatchSession.scala:108)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
  at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
  at scala.collection.TraversableLike$class.map(TraversableLike.scala:244)
  at scala.collection.AbstractTraversable.map(Traversable.scala:105)
  at com.cloudera.livy.sessions.SessionManager.com$cloudera$livy$sessions$SessionManager$$recover(SessionManager.scala:150)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at scala.Option.getOrElse(Option.scala:120)
  at com.cloudera.livy.sessions.SessionManager.<init>(SessionManager.scala:82)
  at com.cloudera.livy.sessions.BatchSessionManager.<init>(SessionManager.scala:42)
  at com.cloudera.livy.server.LivyServer.start(LivyServer.scala:99)
  at com.cloudera.livy.server.LivyServer$.main(LivyServer.scala:302)
  at com.cloudera.livy.server.LivyServer.main(LivyServer.scala)## using "vmstat" found for free memory on the server, we had enough free memory
~~~~

java.lang.OutOfMemoryError: unable to create new native thread highlights; highlights OS cannot assign more native threads to JVMs
Confirmed that this Exception is caused by the violation of the per-process thread count limit

Looking at Livy implementation we found that livy will save the session state in ZK (in HDInsight) and recover those sessions on restart. When restarting, livy will create a thread for each session. 

This is part of High Availability for LivyServer
Refer section High Availability under https://hortonworks.com/blog/livy-a-rest-interface-for-apache-spark/
In case if Livy Server fails, all the connections to  Spark Clusters are also terminated, which means that all the jobs and related data will be lost.
As a session recovery mechanism Livy stores the session details in Zookeeper to be recovered after the livy Server is back.


In this scenario we found customer had submitted a large number of jobs to livy. So it accumulated a certain amount of to-be-recovered sessions causing too many threads being created.

####Mitigation: delete entry, livy/v1/batch , in zk,
 