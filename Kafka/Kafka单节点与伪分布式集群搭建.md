# Kafka单节点与伪分布式集群搭建

参考：简书-《Kafka单节点与伪分布式集群搭建》



------

环境准备

- 服务器集群
  我用的CentOS-6.7版本的4个虚拟机，主机名为hadoop01、hadoop02、hadoop03、hadoop03和hadoop04，其中hadoop01、hadoop02和hadoop03是zookeeper集群，我会在hadoop04中安装kafka，因为在生产环境中，一般把zookeeper集群和kafka集群分机架部署，另外我会使用hadoop用户搭建集群，生产环境中root用户不是可以随意使用的
- 关于虚拟机的安装可以参考以下两篇文章：
  [在Windows中安装一台Linux虚拟机](https://www.jianshu.com/p/da913b9b5216) 
  [通过已有的虚拟机克隆四台虚拟机](https://www.jianshu.com/p/7f2f8a464815)
- Zookeeper集群
  参考[zookeeper-3.4.10的安装配置](https://www.jianshu.com/p/5a4d7390bbfd)
- kafka安装包
  下载地址：[https://mirrors.aliyun.com/apache/kafka/](https://link.jianshu.com/?t=https%3A%2F%2Fmirrors.aliyun.com%2Fapache%2Fkafka%2F) 
  我用的kafka_2.11-0.10.2.1.tgz

------

# 一、Kafka单节点安装部署

## 1. kafka安装包上传到服务器并解压

```
[hadoop@hadoop04 ~]tar -zxvf /opt/soft/kafka_2.11-0.10.2.1.tgz -C /opt/apps/
```

## 2. 进入kafka的config目录下，修改server.properties文件

```
[hadoop@hadoop01 ~]$ cd /opt/apps/kafka_2.11-0.10.2.1/config/
[hadoop@hadoop01 config]$ vim server.properties

# 以下3个配置是需要修改的，其余保持默认即可
broker.id=11
log.dirs=/opt/data/kafka/broker11
zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181
listeners=PLAINTEXT://node1.myexample.com:9092
port=9092
```

说明：

- 只修改列出的3个配置即可，其余保持默认
- broker.id在每个节点上是唯一的，在分布式集群中，有几个机器中安装了Kafka，那么那几个机器中的Kafka的broker.id一定是不同的，在伪分布式集群中，每个server.properties配置文件中的broker.id都是不同的
- log.dirs指定的kafka中的数据的存放位置，默认的tmp目录会定期清空，所以需要修改，而且指定的目录需要在启动kafka集群之前创建好
- zookeeper.connect如果不指定，将使用kafka自带的zookeeper

## 3. 创建log.dirs指定的目录

```
[hadoop@hadoop04 config]$ mkdir -p /opt/data/kafka
```

## 4. 启动zookeeper集群

```
[hadoop@hadoop01 ~]$ zkServer.sh start
[hadoop@hadoop02 ~]$ zkServer.sh start
[hadoop@hadoop03 ~]$ zkServer.sh start
```

## 5. 启动kafka

```
[hadoop@hadoop04 kafka_2.11-0.10.2.1]$ bin/kafka-server-start.sh -daemon config/server.properties

# -daemon选项的意思是后台启动服务
```

## 7. 验证kafka服务是否启动

```
[hadoop@hadoop04 kafka_2.11-0.10.2.1]$ jps
2340 Jps
2286 Kafka
```

## 8. 查看zookeeper中的节点信息

```
[hadoop@hadoop01 ~]$ zkCli.sh
[zk: localhost:2181(CONNECTED) 2] ls /brokers/ids
[11]
```

Kafka 单节点搭建成功！

# 二、Kafka伪分布式安装部署

所谓Kafka伪分布式，就是一个节点启动多个Kafka服务，只需要新增加server.properties配置文件，并按照新的配置文件再启动一个服务即可，当然数量可以看自己心情，我这里就再启动一个kafka服务

## 1. 在config目录下新增加一个server-2.properties文件

```
[hadoop@hadoop04 config]$ cp server.properties server-2.properties 
[hadoop@hadoop04 config]$ vim server-2.properties

broker.id=12
port=9093
log.dirs=/opt/data/kafka/broker12
listeners=PLAINTEXT://node1.myexample.com:9092
port=9092
```

说明：

- broker.id一定要修改
- 新增了port这个配置，指定服务启动占用的端口，上一个配置文件中没有配置，因为默认使用9092端口，上一个服务启动后，9092端口就被占用了，所以这里配置一个新的端口
- log.dirs也需要修改，每个broker应该存放自己的数据，所以需要在配置一下broker12的数据存放路径，启动服务之前先创建好这个目录
- 其余配置和server.properties相同即可

## 2. 再启动一个Kafka服务

```
[hadoop@hadoop04 kafka_2.11-0.10.2.1]$ bin/kafka-server-start.sh -daemon config/server-2.properties
```

注意：启动Kafka集群之前一定要先启动zookeeper集群，我上面已经启动了zookeeper集群，所以这里没有再启

## 3. 验证kafka服务是否启动

```
[hadoop@hadoop04 kafka_2.11-0.10.2.1]$ jps
22485 Jps
22461 Kafka
2286 Kafka
1982 QuorumPeerMain
```

可以看到，启动了两个Kafka服务

## 4. 查看zookeeper中的节点信息

```
[zk: localhost:2181(CONNECTED) 3] ls /brokers/ids
[11, 12]
```

Kafka伪分布式集群搭建成功！



## 5.测试消息是否正常消费

```shell
bin/kafka-console-producer.sh --broker-list 192.168.134.10:9092 --topic topic1
bin/kafka-console-consumer.sh --bootstrap-server 192.168.134.10:9092 --topic topic1 --from-beginning
```

这2行命令用于测试生产消息-消费消息。

在idea中运行kafka的example，如果在消费消息的命令行界面中能看到生产的消息，则说明kafka配置成功。