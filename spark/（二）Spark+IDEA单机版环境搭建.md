# [Spark+IDEA单机版环境搭建](https://www.cnblogs.com/jackchen-Net/p/6623829.html)





文章目录[(?)](https://www.cnblogs.com/jackchen-Net/p/6623829.html#)[[+\]](https://www.cnblogs.com/jackchen-Net/p/6623829.html#)





**要点导航**

- [1. IDEA中配置Spark运行环境](https://www.cnblogs.com/jackchen-Net/p/6623829.html#_label0)

- [2.Spark在windows环境搭建](https://www.cnblogs.com/jackchen-Net/p/6623829.html#_label1)

- [3.其他可能碰到的问题   ](https://www.cnblogs.com/jackchen-Net/p/6623829.html#_label2)

  

[回到导航](https://www.cnblogs.com/jackchen-Net/p/6623829.html#_labelTop)

### 1. IDEA中配置Spark运行环境

​    请参考博文：http://www.cnblogs.com/jackchen-Net/p/6867838.html

**3.1.Project Struct查看项目的配置信息**

 ![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326194431065-2110284598.png)

**3.2.IDEA中如果没有默认安装Scala，可在本地安装即可**

   如果需要安装多版本的scala请注意：

   如果您在本地已经安装了msi结尾的scala，还需要安装第二个版本，建议下载zip包，优点是直接解压在IDEA中配置即可。如第3步所示。

   注意：scala下载地址：<http://www.scala-lang.org/download/2.10.4.html>

**3.3.查看scala环境配置，可以通过下图绿色的”+”添加本地已经下载的scala安装包**

   ![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326194754018-484480840.png)

   ![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326194824080-1946142129.png)

**3.4.特别注意，如果在执行spark代码遇到如下问题，请更改scala版本**

```
Exception in thread "main" java.lang.NoSuchMethodError:
scala.collection.immutable.HashSet$.empty()Lscala/collection/immutable/HashSet;
at akka.actor.ActorCell$.<init>(ActorCell.scala:336)
at akka.actor.ActorCell$.<clinit>(ActorCell.scala)
at akka.actor.RootActorPath.$div(ActorPath.scala:159)
at akka.actor.LocalActorRefProvider.<init>(ActorRefProvider.scala:464)
at akka.actor.LocalActorRefProvider.<init>(ActorRefProvider.scala:452)
at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:39)
at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:27)
at java.lang.reflect.Constructor.newInstance(Constructor.java:513)
at akka.actor.ReflectiveDynamicAccess$$anonfun$createInstanceFor$2.apply(DynamicAccess.scala:78)
at scala.util.Try$.apply(Try.scala:191)
at akka.actor.ReflectiveDynamicAccess.createInstanceFor(DynamicAccess.scala:73)
at akka.actor.ReflectiveDynamicAccess$$anonfun$createInstanceFor$3.apply(DynamicAccess.scala:84)
at akka.actor.ReflectiveDynamicAccess$$anonfun$createInstanceFor$3.apply(DynamicAccess.scala:84)
at scala.util.Success.flatMap(Try.scala:230)
at akka.actor.ReflectiveDynamicAccess.createInstanceFor(DynamicAccess.scala:84)
at akka.actor.ActorSystemImpl.liftedTree1$1(ActorSystem.scala:584)
at akka.actor.ActorSystemImpl.<init>(ActorSystem.scala:577)
at akka.actor.ActorSystem$.apply(ActorSystem.scala:141)
at akka.actor.ActorSystem$.apply(ActorSystem.scala:108)
at akka.Akka$.delayedEndpoint$akka$Akka$1(Akka.scala:11)
at akka.Akka$delayedInit$body.apply(Akka.scala:9)
at scala.Function0$class.apply$mcV$sp(Function0.scala:40)
at scala.runtime.AbstractFunction0.apply$mcV$sp(AbstractFunction0.scala:12)
at scala.App$$anonfun$main$1.apply(App.scala:76)
at scala.App$$anonfun$main$1.apply(App.scala:76)
at scala.collection.immutable.List.foreach(List.scala:383)
at scala.collection.generic.TraversableForwarder$class.foreach(TraversableForwarder.scala:35)
at scala.App$class.main(App.scala:76)
at akka.Akka$.main(Akka.scala:9)
at akka.Akka.main(Akka.scala)
```

　　解决方法是将scala2.11版本改为2.10版本即可。（注意:spark版本为1.6.0）

**3.5.导入程序运行所需要的jar包**

-  通过libary，点击”+”将spark-assembly-1.6.0-hadoop2.6.0.jar导入Classes位置
-  通过spark官网下载spark1.6.0的源码文件(spark1.6.0-src.tgz)解压在windows本地后，通过点击最右侧的”+”导入所有的源码包，从而可以查看源代码。

 ![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326195254565-2131353203.png)

 **3.6.建立一个scala文件并写代码**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.bigdata.demo
import org.apache.spark.{SparkConf, SparkContext}
/**
  * Created by SimonsZhao on 3/25/2017.
  */
object  wordCount {
  def main(args: Array[String]) {
    val conf =new SparkConf().setMaster("local").setAppName("wordCount")
    val sc =new SparkContext(conf)
    val data=sc.textFile("E://scala//spark//testdata//word.txt")
    data.flatMap(_.split("\t")).map((_,1)).reduceByKey(_+_).collect().foreach(println)
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**3.7.运行结果**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
17/03/25 17:25:49 INFO SparkContext: Running Spark version 1.6.0
17/03/25 17:25:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
17/03/25 17:25:50 INFO SecurityManager: Changing view acls to: SimonsZhao
17/03/25 17:25:50 INFO SecurityManager: Changing modify acls to: SimonsZhao
17/03/25 17:25:50 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(SimonsZhao); users with modify permissions: Set(SimonsZhao)
17/03/25 17:25:51 INFO Utils: Successfully started service 'sparkDriver' on port 53279.
17/03/25 17:25:51 INFO Slf4jLogger: Slf4jLogger started
17/03/25 17:25:51 INFO Remoting: Starting remoting
17/03/25 17:25:51 INFO Remoting: Remoting started; listening on addresses :[akka.tcp://sparkDriverActorSystem@192.168.191.1:53292]
17/03/25 17:25:51 INFO Utils: Successfully started service 'sparkDriverActorSystem' on port 53292.
17/03/25 17:25:51 INFO SparkEnv: Registering MapOutputTracker
17/03/25 17:25:51 INFO SparkEnv: Registering BlockManagerMaster
17/03/25 17:25:51 INFO DiskBlockManager: Created local directory at C:\Users\SimonsZhao\AppData\Local\Temp\blockmgr-7e548732-b1db-4e3c-acdb-37e686b10dff
17/03/25 17:25:51 INFO MemoryStore: MemoryStore started with capacity 2.4 GB
17/03/25 17:25:51 INFO SparkEnv: Registering OutputCommitCoordinator
17/03/25 17:25:51 INFO Utils: Successfully started service 'SparkUI' on port 4040.
17/03/25 17:25:51 INFO SparkUI: Started SparkUI at http://192.168.191.1:4040
17/03/25 17:25:52 INFO Executor: Starting executor ID driver on host localhost
17/03/25 17:25:52 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 53299.
17/03/25 17:25:52 INFO NettyBlockTransferService: Server created on 53299
17/03/25 17:25:52 INFO BlockManagerMaster: Trying to register BlockManager
17/03/25 17:25:52 INFO BlockManagerMasterEndpoint: Registering block manager localhost:53299 with 2.4 GB RAM, BlockManagerId(driver, localhost, 53299)
17/03/25 17:25:52 INFO BlockManagerMaster: Registered BlockManager
17/03/25 17:25:52 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 153.6 KB, free 153.6 KB)
17/03/25 17:25:52 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 13.9 KB, free 167.5 KB)
17/03/25 17:25:52 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on localhost:53299 (size: 13.9 KB, free: 2.4 GB)
17/03/25 17:25:52 INFO SparkContext: Created broadcast 0 from textFile at wordCount.scala:11
17/03/25 17:25:54 WARN : Your hostname, SimonsCJ resolves to a loopback/non-reachable address: fe80:0:0:0:0:5efe:c0a8:bf01%30, but we couldn't find any external IP address!
17/03/25 17:25:55 INFO FileInputFormat: Total input paths to process : 1
17/03/25 17:25:55 INFO SparkContext: Starting job: collect at wordCount.scala:12
17/03/25 17:25:55 INFO DAGScheduler: Registering RDD 3 (map at wordCount.scala:12)
17/03/25 17:25:55 INFO DAGScheduler: Got job 0 (collect at wordCount.scala:12) with 1 output partitions
17/03/25 17:25:55 INFO DAGScheduler: Final stage: ResultStage 1 (collect at wordCount.scala:12)
17/03/25 17:25:55 INFO DAGScheduler: Parents of final stage: List(ShuffleMapStage 0)
17/03/25 17:25:55 INFO DAGScheduler: Missing parents: List(ShuffleMapStage 0)
17/03/25 17:25:55 INFO DAGScheduler: Submitting ShuffleMapStage 0 (MapPartitionsRDD[3] at map at wordCount.scala:12), which has no missing parents
17/03/25 17:25:55 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 4.1 KB, free 171.6 KB)
17/03/25 17:25:55 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 2.3 KB, free 173.9 KB)
17/03/25 17:25:55 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on localhost:53299 (size: 2.3 KB, free: 2.4 GB)
17/03/25 17:25:55 INFO SparkContext: Created broadcast 1 from broadcast at DAGScheduler.scala:1006
17/03/25 17:25:55 INFO DAGScheduler: Submitting 1 missing tasks from ShuffleMapStage 0 (MapPartitionsRDD[3] at map at wordCount.scala:12)
17/03/25 17:25:55 INFO TaskSchedulerImpl: Adding task set 0.0 with 1 tasks
17/03/25 17:25:55 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, partition 0,PROCESS_LOCAL, 2129 bytes)
17/03/25 17:25:55 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
17/03/25 17:25:55 INFO HadoopRDD: Input split: file:/E:/scala/spark/testdata/word.txt:0+19
17/03/25 17:25:55 INFO deprecation: mapred.tip.id is deprecated. Instead, use mapreduce.task.id
17/03/25 17:25:55 INFO deprecation: mapred.task.id is deprecated. Instead, use mapreduce.task.attempt.id
17/03/25 17:25:55 INFO deprecation: mapred.task.is.map is deprecated. Instead, use mapreduce.task.ismap
17/03/25 17:25:55 INFO deprecation: mapred.task.partition is deprecated. Instead, use mapreduce.task.partition
17/03/25 17:25:55 INFO deprecation: mapred.job.id is deprecated. Instead, use mapreduce.job.id
17/03/25 17:25:55 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 2253 bytes result sent to driver
17/03/25 17:25:55 INFO DAGScheduler: ShuffleMapStage 0 (map at wordCount.scala:12) finished in 0.228 s
17/03/25 17:25:55 INFO DAGScheduler: looking for newly runnable stages
17/03/25 17:25:55 INFO DAGScheduler: running: Set()
17/03/25 17:25:55 INFO DAGScheduler: waiting: Set(ResultStage 1)
17/03/25 17:25:55 INFO DAGScheduler: failed: Set()
17/03/25 17:25:55 INFO DAGScheduler: Submitting ResultStage 1 (ShuffledRDD[4] at reduceByKey at wordCount.scala:12), which has no missing parents
17/03/25 17:25:55 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 194 ms on localhost (1/1)
17/03/25 17:25:55 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool
17/03/25 17:25:55 INFO MemoryStore: Block broadcast_2 stored as values in memory (estimated size 2.6 KB, free 176.4 KB)
17/03/25 17:25:55 INFO MemoryStore: Block broadcast_2_piece0 stored as bytes in memory (estimated size 1600.0 B, free 178.0 KB)
17/03/25 17:25:55 INFO BlockManagerInfo: Added broadcast_2_piece0 in memory on localhost:53299 (size: 1600.0 B, free: 2.4 GB)
17/03/25 17:25:55 INFO SparkContext: Created broadcast 2 from broadcast at DAGScheduler.scala:1006
17/03/25 17:25:55 INFO DAGScheduler: Submitting 1 missing tasks from ResultStage 1 (ShuffledRDD[4] at reduceByKey at wordCount.scala:12)
17/03/25 17:25:55 INFO TaskSchedulerImpl: Adding task set 1.0 with 1 tasks
17/03/25 17:25:55 INFO TaskSetManager: Starting task 0.0 in stage 1.0 (TID 1, localhost, partition 0,NODE_LOCAL, 1894 bytes)
17/03/25 17:25:55 INFO Executor: Running task 0.0 in stage 1.0 (TID 1)
17/03/25 17:25:55 INFO ShuffleBlockFetcherIterator: Getting 1 non-empty blocks out of 1 blocks
17/03/25 17:25:55 INFO ShuffleBlockFetcherIterator: Started 0 remote fetches in 6 ms
17/03/25 17:25:55 INFO Executor: Finished task 0.0 in stage 1.0 (TID 1). 1349 bytes result sent to driver
17/03/25 17:25:55 INFO DAGScheduler: ResultStage 1 (collect at wordCount.scala:12) finished in 0.059 s
17/03/25 17:25:55 INFO TaskSetManager: Finished task 0.0 in stage 1.0 (TID 1) in 59 ms on localhost (1/1)
17/03/25 17:25:55 INFO TaskSchedulerImpl: Removed TaskSet 1.0, whose tasks have all completed, from pool
17/03/25 17:25:55 INFO DAGScheduler: Job 0 finished: collect at wordCount.scala:12, took 0.532461 s
(you,1)
(hello,2)
(me,1)
17/03/25 17:25:55 INFO SparkContext: Invoking stop() from shutdown hook
17/03/25 17:25:56 INFO SparkUI: Stopped Spark web UI at http://192.168.191.1:4040
17/03/25 17:25:56 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
17/03/25 17:25:56 INFO MemoryStore: MemoryStore cleared
1/03/25 17:25:56 INFO BlockManager: BlockManager stopped
17/03/25 17:25:56 INFO BlockManagerMaster: BlockManagerMaster stopped
17/03/25 17:25:56 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
17/03/25 17:25:56 INFO SparkContext: Successfully stopped SparkContext
17/03/25 17:25:56 INFO ShutdownHookManager: Shutdown hook called
17/03/25 17:25:56 INFO ShutdownHookManager: Deleting directory C:\Users\SimonsZhao\AppData\Local\Temp\spark-220c67fe-f2c3-400b-bfe1-fe833e33e74f
```