# Flume--集群及项目实战

[Flume](https://blog.xiaoxiaomo.com/tags/Flume/)             

1. [Flume 集群](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#Flume-集群)
2. [常见架构](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#常见架构)
3. 实战
   1. [需求](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#需求)
   2. [画图](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#画图)
   3. [准备](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#准备)
   4. [C机器](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#C机器)
   5. [A机器](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#A机器)
   6. [验证功能](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#验证功能)
   7. [同步节点](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#同步节点)
4. [博客源码](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#博客源码)

　　本篇博客主要讲解**flume集群的搭建及项目实战**，flume集群相对来说比较简单，重点是后面的项目实战。如果对**flume**还不够理解或者它的**组件**不熟悉可以阅读上篇博客：<http://blog.xiaoxiaomo.com/2016/05/22/Flume-日志收集/>

## Flume 集群

1. **解压缩** ： tar -zxvf [apache-flume-1.6.0-bin.tar.gz](https://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/href=%22http://www.apache.org/dyn/closer.lua/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz%22) -C /opt/ ;
2. **重命名** ： mv /opt/apache-flume-1.6.0-bin/ /opt/flume（可省略） ;
3. **复制配置文件** ： cp conf/flume-env.sh.template conf/flume-env.sh ;
4. **修改conf/flume-env.sh** : JAVA_HOME ;
5. **复制flume到其他节点** ： scp -r …… 。



## 常见架构

- 常见架构

  ![agent到另一个agent](https://img.xiaoxiaomo.com/blog/img/20160522191722.jpg)

  ![整合多个agent，最后汇总](https://img.xiaoxiaomo.com/blog/img/20160522191750.jpg)

  ![多路复用](https://img.xiaoxiaomo.com/blog/img/20160522191808.jpg)

## 实战

### 需求

- A、B两台机器实时生产日志主要类型为`access.log`、`ugcheader.log`、`ugctail.log` , **要求**：

1. 把A、B 机器中的access.log、ugcheader.log、ugctail.log 汇总到C机器上然后统一收集到hdfs和Kafka中。
2. 在hdfs中要求的目录为：用作离线统计。
   **/source/access/2016-01-01/**
   **/source/ugcheader/2016-01-01/**
   **/source/ugctail/2016-01-01/**
3. Kafka分topic , 用作实时分析。

### 画图



![项目结构图](https://img.xiaoxiaomo.com/blog/img/20160522195635.jpg)



### 准备

- **机器** (*博主使用了三台机器*)
  A机器 : xxo 08 安装 ： [zookeeper](http://blog.xiaoxiaomo.com/2016/05/05/Zookeeper-%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/) 、 [kafka](http://blog.xiaoxiaomo.com/2016/05/14/Kafka-%E9%9B%86%E7%BE%A4%E5%8F%8AAPI%E6%93%8D%E4%BD%9C/) 、[flume](http://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#Flume-集群)
  B机器 : xxo 09 安装 ： [zookeeper](http://blog.xiaoxiaomo.com/2016/05/05/Zookeeper-%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/) 、 [kafka](http://blog.xiaoxiaomo.com/2016/05/14/Kafka-%E9%9B%86%E7%BE%A4%E5%8F%8AAPI%E6%93%8D%E4%BD%9C/) 、[flume](http://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#Flume-集群)
  C机器 : xxo 10 安装 ： [zookeeper](http://blog.xiaoxiaomo.com/2016/05/05/Zookeeper-%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/) 、 [kafka](http://blog.xiaoxiaomo.com/2016/05/14/Kafka-%E9%9B%86%E7%BE%A4%E5%8F%8AAPI%E6%93%8D%E4%BD%9C/) 、[flume](http://blog.xiaoxiaomo.com/2016/05/22/Flume-%E9%9B%86%E7%BE%A4%E5%8F%8A%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/#Flume-集群) 、[hadoop伪分布式](http://blog.xiaoxiaomo.com/2016/05/08/Hadoop-2-0%E4%BC%AA%E5%88%86%E5%B8%83%E5%BC%8F%E5%AE%89%E8%A3%85/)
- **启动**

1. 启动zookeeper : /opt/zookeeper/bin/zkServer.sh start

2. 启动kafka : nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties >>/opt/logs/kafka-server.log 2>&1 &

3. 启动hdfs : start-dfs.sh 

4. 这里查看一下xxo10的进程 : 

   ```
   [root@xxo10 flume]# jps
   1305 Kafka
   1252 QuorumPeerMain
   1786 Jps
   1542 DataNode
   1454 NameNode
   ```

- 创建topic

  ```
  [root@xxo08 kafka]# bin/kafka-topics.sh --create --zookeeper xxo08:2181,xxo09:2181,xxo10:2181 --replication-factor 3 --partition 3  --topic access 
  [root@xxo08 kafka]# bin/kafka-topics.sh --create --zookeeper xxo08:2181,xxo09:2181,xxo10:2181 --replication-factor 3 --partition 3  --topic ugchead 
  [root@xxo08 kafka]# bin/kafka-topics.sh --create --zookeeper xxo08:2181,xxo09:2181,xxo10:2181 --replication-factor 3 --partition 3  --topic ugctail 
  [root@xxo08 kafka]# bin/kafka-topics.sh --list --zookeeper xxo08:2181,xxo09:2181,xxo10:2181   ###查看
  access
  ugchead
  ugctail
  ```

### C机器

```
[root@xxo10 flume]# vim conf/hdfs_kafka.conf

#################################### C机器 #########################################
#################################### 两个channel、两个sink ##########################

# Name the components on this agent
a1.sources = r1
a1.sinks = kfk fs
a1.channels = c1 c2

# varo source
a1.sources.r1.type = avro
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 44444

# source r1定义拦截器，为消息添加时间戳
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.apache.flume.interceptor.TimestampInterceptor$Builder

# kfk sink
a1.sinks.kfk.type = org.apache.flume.sink.kafka.KafkaSink
#a1.sinks.kfk.topic = mytopic
a1.sinks.kfk.brokerList = xxo08:9092,xxo09:9092,xxo10:9092


# hdfs sink
a1.sinks.fs.type = hdfs
a1.sinks.fs.hdfs.path = hdfs://xxo10:9000/source/%{type}/%Y%m%d
a1.sinks.fs.hdfs.filePrefix = events-
a1.sinks.fs.hdfs.fileType = DataStream
#a1.sinks.fs.hdfs.fileType = CompressedStream
#a1.sinks.fs.hdfs.codeC = gzip
#不按照条数生成文件
a1.sinks.fs.hdfs.rollCount = 0
#如果压缩存储的话HDFS上的文件达到64M时生成一个文件注意是压缩前大小为64生成一个文件，然后压缩存储。
a1.sinks.fs.hdfs.rollSize = 67108864
a1.sinks.fs.hdfs.rollInterval = 0


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 1000

a1.channels.c2.type = memory
a1.channels.c2.capacity = 10000
a1.channels.c2.transactionCapacity = 1000

# Bind the source and sink to the channel
a1.sources.r1.channels = c1 c2
a1.sinks.kfk.channel = c1
a1.sinks.fs.channel = c2
```

启动

```
[root@xxo10 ~]# cd /opt/apache-flume/
[root@xxo10 flume]# bin/flume-ng agent --conf conf/ --conf-file conf/hdfs_kafka.conf --name a1 -Dflume.root.logger=INFO,console &
......
......
......Component type: SINK, name: kfk started  ##启动成功
```

### **A机器**

```
[root@xxo08 flume]# vim conf/hdfs_kafka.conf

#################################### A机器 #########################################
#################################### 3个source #####################################
#################################### 2个拦截器 ######################################
# Name the components on this agent
a1.sources = access ugchead ugctail
a1.sinks = k1
a1.channels = c1

# 三个sources
a1.sources.access.type = exec
a1.sources.access.command = tail -F /root/data/access.log

a1.sources.ugchead.type = exec
a1.sources.ugchead.command = tail -F /root/data/ugchead.log

a1.sources.ugctail.type = exec
a1.sources.ugctail.command = tail -F /root/data/ugctail.log

# sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = xxo10
a1.sinks.k1.port = 44444

# channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 1000

# interceptor
a1.sources.access.interceptors = i1 i2
a1.sources.access.interceptors.i1.type=static
a1.sources.access.interceptors.i1.key = type
a1.sources.access.interceptors.i1.value = access
a1.sources.access.interceptors.i2.type=static
a1.sources.access.interceptors.i2.key = topic
a1.sources.access.interceptors.i2.value = access

a1.sources.ugchead.interceptors = i1 i2
a1.sources.ugchead.interceptors.i1.type=static
a1.sources.ugchead.interceptors.i1.key = type
a1.sources.ugchead.interceptors.i1.value = ugchead
a1.sources.ugchead.interceptors.i2.type=static
a1.sources.ugchead.interceptors.i2.key = topic
a1.sources.ugchead.interceptors.i2.value = ugchead

a1.sources.ugctail.interceptors = i1 i2
a1.sources.ugctail.interceptors.i1.type=static
a1.sources.ugctail.interceptors.i1.key = type
a1.sources.ugctail.interceptors.i1.value = ugctail
a1.sources.ugctail.interceptors.i2.type=static
a1.sources.ugctail.interceptors.i2.key = topic
a1.sources.ugctail.interceptors.i2.value = ugctail

# Bind the source and sink to the channel
a1.sources.access.channels = c1
a1.sources.ugchead.channels = c1
a1.sources.ugctail.channels = c1
a1.sinks.k1.channel = c1
```

启动A机器

```
[root@xxo08 flume]# bin/flume-ng agent --conf conf/ --conf-file conf/hdfs_kafka.conf --name a1 -Dflume.root.logger=INFO,console &
```

### 验证功能

我这里启动了一个向access.log`、`ugcheader.log`、`ugctail.log`添加数据的java程序:

```
[root@xxo08 ~]# java -cp KafkaFlumeProject_20160519-1.0-SNAPSHOT-jar-with-dependencies.jar com.xxo.utils.Creator
```

1. 查看**hdfs**的情况

   ```
   [root@xxo10 ~]# hdfs dfs -text /source/ugchead/20160523/* | more 
   1001	221.8.9.6 80	be83f3fd-a218-4f98-91d8-6b4f0bb4558b	750b6203-4a7d-42d5-82e4-906415b70f63	10207	{"ugctype":"consumer",
   "userId":"40604","coin":"10","number":"2"}	1463685721663
   1003	218.75.100.114	ea11f1d2-680d-4645-a52e-74d5f2317dfd	8109eda1-aeac-43fe-94b1-85d2d1934913	20101	{"ugctype":"fav","user
   Id":"40604","item":"13"}	1463685722666
   ......
   ```

2. **kafka**消费者

   ```
   ########################## 这里查看一下access ###############################
   [root@xxo09 ~]# /opt/kafka/bin/kafka-console-consumer.sh --zookeeper xxo08:2181,xxo09:2181,xxo10:2181 --topic access  --from-beginning 
   1001	218.26.219.186	070c8525-b857-414d-98b6-13134da08401	10201	0	GET /tologin HTTP/1.1	408	/update/pass	Mozilla/5.0 (Windows; U; Windows NT 5.1)Gecko/20070803 Firefox/1.5.0.12	1463676319717
   ......
   ```

3. 查看日志：
   tac /opt/flume/logs/flume.log | more

### 同步节点

- 【B机器】

  ```
  ####################### 拷贝 ##################################
  [root@xxo08 flume]# scp /opt/flume/conf/hdfs_kafka.conf root@xxo09:/opt/flume/conf/
  hdfs_kafka.conf              100% 1803     1.8KB/s   00:00 
  
  ####################### 远程启动 ###############################
  [root@xxo08 flume]# ssh root@xxo09 "/opt/flume/bin/flume-ng agent --conf /opt/flume/conf/ --conf-file /opt/flume/conf/hdfs_kafka.conf --name a1" &
  ```

## 博客源码

- 本例中使用的配置源码下载：
  <https://github.com/jasonTangxd/Blog_Resources_20160508/tree/master/resources/flume/project>

[上一篇：Flume--负载均衡和故障转移   ](https://blog.xiaoxiaomo.com/2016/05/23/Flume-负载均衡和故障转移/) 

[    下一篇：Flume--日志收集 ](https://blog.xiaoxiaomo.com/2016/05/22/Flume-日志收集/)