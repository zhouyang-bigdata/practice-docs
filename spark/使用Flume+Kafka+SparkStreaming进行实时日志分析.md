

#  使用Flume+Kafka+SparkStreaming进行实时日志分析                  

目录(?)[+]

1. [整体架构](http://blog.csdn.net/trigl/article/details/70237981#t0)

2. [实战演练](http://blog.csdn.net/trigl/article/details/70237981#t1)

3. 1. [1 安装Kafka](http://blog.csdn.net/trigl/article/details/70237981#t2)
   2. [2 安装Flume](http://blog.csdn.net/trigl/article/details/70237981#t3)
   3. [3 SparkStreaming编程](http://blog.csdn.net/trigl/article/details/70237981#t4)



每个公司想要进行数据分析或数据挖掘，收集日志、ETL都是第一步的，今天就讲一下如何实时地（准实时，每分钟分析一次）收集日志，处理日志，把处理后的记录存入[Hive](http://lib.csdn.net/base/hive)中，并附上完整实战代码

# 1.  整体架构

思考一下，正常情况下我们会如何收集并分析日志呢？

首先，业务日志会通过Nginx（或者其他方式，我们是使用Nginx写入日志）每分钟写入到磁盘中，现在我们想要使用[Spark](http://lib.csdn.net/base/spark)分析日志，就需要先将磁盘中的文件上传到HDFS上，然后Spark处理，最后存入[hive](http://lib.csdn.net/base/hive)表中，如图所示：

![这里写图片描述](file:///C:/Users/zy/OneDrive/aa开发资料2017.06.06/Spark实例/Flume+kafka+spark%20streaming/使用Flume+Kafka+SparkStreaming进行实时日志分析%20-%20Trigl的博客%20-%20CSDN博客_files/20170518173937190)

我们之前就是使用这种方式每天分析一次日志，但是这样有几个缺点：

首先我们的日志是通过Nginx每分钟存成一个文件，这样一天的文件数很多，不利于后续的分析任务，所以先要把一天的所有日志文件合并起来

合并起来以后需要把该文件从磁盘传到Hdfs上，但是我们的日志服务器并不在[Hadoop](http://lib.csdn.net/base/hadoop)集群内，所以没办法直接传到Hdfs上，需要首先把文件从日志服务器传输到[hadoop](http://lib.csdn.net/base/hadoop)集群所在的服务器，然后再上传到Hdfs

最后也是最重要的，滞后一天分析数据已经不能满足我们新的业务需求了，最好能控制在一个小时的滞后时间

可以看出来我们以前收集分析日志的方式还是比较原始的，而且比较耗时，很多时间浪费在了网络传输上面，如果日志量大的话还有丢失数据的可能性，所以在此基础上改进了一下[架构](http://lib.csdn.net/base/architecture)：

![这里写图片描述](file:///C:/Users/zy/OneDrive/aa开发资料2017.06.06/Spark实例/Flume+kafka+spark%20streaming/使用Flume+Kafka+SparkStreaming进行实时日志分析%20-%20Trigl的博客%20-%20CSDN博客_files/20170518175830889)

整个过程就是，Flume会实时监控写入日志的磁盘，只要有新的日志写入，Flume就会将日志以消息的形式传递给Kafka，然后Spark Streaming实时消费消息传入Hive

那么Flume是什么呢，它为什么可以监控一个磁盘文件呢？简而言之，Flume是用来收集、汇聚并且移动大量日志文件的开源框架，所以很适合这种实时收集日志并且传递日志的场景

Kafka是一个消息系统，Flume收集的日志可以移动到Kafka消息队列中，然后就可以被多处消费了，而且可以保证不丢失数据

通过这套架构，收集到的日志可以及时被Flume发现传到Kafka，通过Kafka我们可以把日志用到各个地方，同一份日志可以存入Hdfs中，也可以离线进行分析，还可以实时计算，而且可以保证安全性，基本可以达到实时的要求

整个流程已经清晰了，下面各个突破，我们开始动手实现整套系统

# 2.  实战演练

## 2.1 安装Kafka

下载安装Kafka以及一些基本命令请传送到这里：[ Kafka安装与简介 ](http://blog.csdn.net/trigl/article/details/72581735)

安装好以后新建名为launcher_click的topic：

```
bin/kafka-topics.sh --create --zookeeper hxf:2181,cfg:2181,jqs:2181,jxf:2181,sxtb:2181 --replication-factor 2 --partitions 2 --topic launcher_click1
1
```

查看一下该topic：

```
bin/kafka-topics.sh --describe --zookeeper hxf:2181,cfg:2181,jqs:2181,jxf:2181,sxtb:2181 --topic launcher_click1
1
```

![这里写图片描述](file:///C:/Users/zy/OneDrive/aa开发资料2017.06.06/Spark实例/Flume+kafka+spark%20streaming/使用Flume+Kafka+SparkStreaming进行实时日志分析%20-%20Trigl的博客%20-%20CSDN博客_files/20170524141002525)

## 2.2 安装Flume

1、下载解压

下载地址： <https://flume.apache.org/download.html> 
注意进入下载地址页面，使用清华大学的那个地址，否则会很慢

```
wget http://apache.fayea.com/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
tar -xvf apache-flume-1.7.0-bin.tar.gz12
12
```

2、修改配置文件

进入flume目录，修改conf/flume-env.sh 

```
export JAVA_HOME=/data/install/jdk
export JAVA_OPTS="-Xms1000m -Xmx2000m -Dcom.sun.management.jmxremote"12
12
```

添加配置文件：conf/flume_launcherclick.conf

```
# logser可以看做是flume服务的名称，每个flume都由sources、channels和sinks三部分组成
# sources可以看做是数据源头、channels是中间转存的渠道、sinks是数据后面的去向
logser.sources = src_launcherclick
logser.sinks = kfk_launcherclick
logser.channels = ch_launcherclick

# source
# 源头类型是TAILDIR，就可以实时监控以追加形式写入文件的日志
logser.sources.src_launcherclick.type = TAILDIR
# positionFile记录所有监控的文件信息
logser.sources.src_launcherclick.positionFile = /data/install/flume/position/launcherclick/taildir_position.json
# 监控的文件组
logser.sources.src_launcherclick.filegroups = f1
# 文件组包含的具体文件，也就是我们监控的文件
logser.sources.src_launcherclick.filegroups.f1 = /data/launcher/stat_app/.*

# interceptor
# 写kafka的topic即可
logser.sources.src_launcherclick.interceptors = i1 i2
logser.sources.src_launcherclick.interceptors.i1.type=static
logser.sources.src_launcherclick.interceptors.i1.key = type
logser.sources.src_launcherclick.interceptors.i1.value = launcher_click
logser.sources.src_launcherclick.interceptors.i2.type=static
logser.sources.src_launcherclick.interceptors.i2.key = topic
logser.sources.src_launcherclick.interceptors.i2.value = launcher_click

# channel
logser.channels.ch_launcherclick.type = memory
logser.channels.ch_launcherclick.capacity = 10000
logser.channels.ch_launcherclick.transactionCapacity = 1000

# kfk sink
# 指定sink类型是Kafka，说明日志最后要发送到Kafka
logser.sinks.kfk_launcherclick.type = org.apache.flume.sink.kafka.KafkaSink
# Kafka broker
logser.sinks.kfk_launcherclick.brokerList = 10.0.0.80:9092,10.0.0.140:9092

# Bind the source and sink to the channel
logser.sources.src_launcherclick.channels = ch_launcherclick
logser.sinks.kfk_launcherclick.channel = ch_launcherclick12345678910111213141516171819202122232425262728293031323334353637383940
12345678910111213141516171819202122232425262728293031323334353637383940
```

3、启动

```
nohup bin/flume-ng agent --conf conf/ --conf-file conf/flume_launcherclick.conf --name logser -Dflume.root.logger=INFO,console >> logs/flume_launcherclick.log &1
1
```

此时Kafka和Flume都已经启动了，从配置可以看到Flume的监控文件是/data/launcher/stat_app/.*，所以只要该目录下文件内容有增加就会发送到Kafka，大家可以自己追加一些[测试](http://lib.csdn.net/base/softwaretest)日志到这个目录的文件下，然后开一个Kafka Consumer看一下Kafka是否接收到消息，这里我们完成SparkStreaming以后再看测试结果

## 2.3 SparkStreaming编程

SparkStreaming是Spark用来处理实时流的，能够实时到秒级，我们这里不需要这么实时，是每分钟执行一次日志分析程序，主要代码如下：

```
  def main(args: Array[String]) {

    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    System.setProperty("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    val sparkConf = new SparkConf().setAppName("LauncherStreaming")

    //每60秒一个批次
    val ssc = new StreamingContext(sparkConf, Seconds(60))

    // 从Kafka中读取数据
    val kafkaStream = KafkaUtils.createStream(
      ssc,
      "hxf:2181,cfg:2181,jqs:2181,jxf:2181,sxtb:2181", // Kafka集群使用的zookeeper
      "launcher-streaming", // 该消费者使用的group.id
      Map[String, Int]("launcher_click" -> 0, "launcher_click" -> 1), // 日志在Kafka中的topic及其分区
      StorageLevel.MEMORY_AND_DISK_SER).map(_._2) // 获取日志内容

    kafkaStream.foreachRDD((rdd: RDD[String], time: Time) => {
      val result = rdd.map(log => parseLog(log)) // 分析处理原始日志
        .filter(t => StringUtils.isNotBlank(t._1) && StringUtils.isNotBlank(t._2))
      // 存入hdfs
      result.saveAsHadoopFile(HDFS_DIR, classOf[String], classOf[String], classOf[LauncherMultipleTextOutputFormat[String, String]])
    })

    ssc.start()
    // 等待实时流
    ssc.awaitTermination()
  }
```

然后打包上传到master运行：

```
nohup /data/install/spark-2.0.0-bin-hadoop2.7/bin/spark-submit  --master spark://hxf:7077  --executor-memory 1G --total-executor-cores 4   --class com.analysis.main.LauncherStreaming --jars /home/hadoop/jar/kafka-clients-0.10.0.0.jar,/home/hadoop/jar/metrics-core-2.2.0.jar,/home/hadoop/jar/zkclient-0.3.jar,/home/hadoop/jar/spark-streaming-kafka-0-8_2.11-2.0.0.jar,/home/hadoop/jar/kafka_2.11-0.8.2.1.jar  /home/hadoop/jar/SparkLearning.jar  >> /home/hadoop/logs/LauncherDM.log &
```



然后开始测试，往Flume监控目录/data/launcher/stat_app/.*写日志，原始日志内容类似下面这样：

```
118.120.102.3|1495608541.238|UEsDBBQACAgIACB2uEoAAAAAAAAAAAAAAAABAAAAMGWUbW7bMAyGb6NfnUFRFEWhJ+gBdgBZVjpjjp04brMAO*yY2DKa9Y+B1+DnQ1LCztoITgK4wPGHfNUhmKGUPOn3DyP*zdOxSWM3T33XXMqy9OP7xXTZiTC1xlL0HgMEi+BfHoooBEGKr3fPpYy5jMse4Xzupus4TKkrs4kZOhI51CgWWKxsUQBRPMDr1*w5Hcuc0LiUEFBwdXQxAARXHb3+QXlOfzya0uZWOGwlEwBDwLD5oJBVFHsEEPF2U0EUToyr8k4tg9v8AkRrIcKmxGsU2eqQIM45dKuKFICo5oveEqOjh2JAIITImyIJqBk3JS4qh7Wby*TroxnL9ZKHXrsyWeBQoMXaEgXUKh6mOQ1l7NLc*Hwz8aDpAtndLFJEetkVc6S9V*bg+RFiKMvnTv6ahuGUTmWexqEfi3Elezx0botJrCCQn5jfCzWaqaUOqNpFYO23ckYl5GOlx4rLQuUllh27SsjZyLQTUn4K+3uVczlOi+7uuMzTYLoibeIspk71DtKuJC+7T5qXPg9lLddaZs6+Lolnj7ANW0dBGKOn72m3cbQJI2Kq4*C6Xhz9E5Pzeeg*i2l1IAJtpReILNq6DY4peFjHeO5vffPZd2UyejEJ28Puo0sI*2*5ojvhfNcquWomFMVp02Pz++M6Nach3e6XR5wOlrdSg4T7RkgtQAuC6HYl2sc62i6dUq*om+HWjvdHAPSk8hYkegHraxC8PwPons73XZeozDfXmaRzzzaD2XI4fX0QX*8BUEsHCKeftc48AgAAmQQAAA==1
1
```

查看HDFS的对应目录是否有内容：

![这里写图片描述](file:///C:/Users/zy/OneDrive/aa开发资料2017.06.06/Spark实例/Flume+kafka+spark%20streaming/使用Flume+Kafka+SparkStreaming进行实时日志分析%20-%20Trigl的博客%20-%20CSDN博客_files/20170524145206940)

HDFS存储的分析后的日志内容如下：

```
99000945863664;864698037273329|119.176.140.248|1495594615129|2017-05-24 10:56:55|xiaomi|redmi4x|com.jingdong.app.mall&0ae359b6&1495534579412&1;com.autonavi.minimap&279f562f&1495534597934,1495534616627&2;com.android.contacts&91586932&1495538267103,1495540527138,1495576834653,1495583404117,1495591231535&51
1
```

SparkStreaming任务状态如下：

![这里写图片描述](file:///C:/Users/zy/OneDrive/aa开发资料2017.06.06/Spark实例/Flume+kafka+spark%20streaming/使用Flume+Kafka+SparkStreaming进行实时日志分析%20-%20Trigl的博客%20-%20CSDN博客_files/20170524145340469)

可以看到的确是每分钟执行一次

代码地址：

https://github.com/Trigl/SparkLearning/blob/master/src/main/scala/com/trigl/spark/streaming/LauncherStreaming.scala