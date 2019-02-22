# Spark2.1.0入门：Apache Kafka作为DStream数据源



Kafka是非常流行的日志采集系统，可以作为DStream的高级数据源。

## Kafka的安装和准备工作

在安装的时候，要注意，到Kafka官网下载安装文件时，一定要选择和自己电脑上已经安装的scala版本号一致才可以，本教程安装的Spark版本号是2.1.0，scala版本号是2.11，所以，一定要选择Kafka版本号是2.11开头的。比如，到Kafka官网中，可以下载安装文件Kafka_2.11-0.10.2.0.tgz，前面的2.11就是支持的scala版本号，后面的0.10.2.0是Kafka自身的版本号。
这里假设你已经根据这篇博客文章安装成功了Kafka。
下面，我们启动Kafka。
请登录Linux系统（本教程统一使用hadoop用户登录），打开一个终端，输入下面命令启动Zookeeper服务：

```bash
cd /usr/local/kafka./bin/zookeeper-server-start.sh config/zookeeper.properties
```

Shell 命令

注意，执行上面命令以后，终端窗口会返回一堆信息，然后就停住不动了，没有回到shell命令提示符状态，这时，千万不要错误认为死机了，而是Zookeeper服务器启动了，正在处于服务状态。所以，千万不要关闭这个终端窗口，一旦关闭，zookeeper服务就停止了，所以，不能关闭这个终端窗口。
请另外打开第二个终端，然后输入下面命令启动Kafka服务：

```bash
cd /usr/local/kafkabin/kafka-server-start.sh config/server.properties
```

Shell 命令

同样，执行上面命令以后，终端窗口会返回一堆信息，然后就停住不动了，没有回到shell命令提示符状态，这时，千万不要错误认为死机了，而是Kafka服务器启动了，正在处于服务状态。所以，千万不要关闭这个终端窗口，一旦关闭，Kafka服务就停止了，所以，不能关闭这个终端窗口。

当然了，还有一种方式是，采用下面加了“&”的命令:

```bash
cd /usr/local/kafkabin/kafka-server-start.sh config/server.properties &
```

Shell 命令

这样，Kafka就会在后台运行，即使你关闭了这个终端，Kafka也会一直在后台运行。不过，这样做，有时候我们往往就忘记了还有Kafa在后台运行，所以，建议暂时不要用&。

下面先测试一下Kafka是否可以正常使用。再另外打开第三个终端，然后输入下面命令创建一个自定义名称为“wordsendertest”的topic（关于什么是topic，请参考厦门大学数据库实验室博客文章《[Kafka的安装和简单实例测试](http://dblab.xmu.edu.cn/blog/1096-2/)》）：

```bash
cd /usr/local/kafka./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic wordsendertest//这个topic叫wordsendertest，2181是zookeeper默认的端口号，partition是topic里面的分区数，replication-factor是备份的数量，在kafka集群中使用，这里单机版就不用备份了//可以用list列出所有创建的topics,来查看上面创建的topic是否存在./bin/kafka-topics.sh --list --zookeeper localhost:2181
```

Shell 命令

这个名称为“wordsendertest”的topic，就是专门负责采集发送一些单词的。
下面，我们需要用producer来产生一些数据，请在当前终端内继续输入下面命令：

```bash
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic wordsendertest
```

Shell 命令

上面命令执行后，你就可以在当前终端内用键盘输入一些英文单词，比如我们可以输入：

```
hello hadoop
hello spark
```

这些单词就是数据源，这些单词会被Kafka捕捉到以后发送给消费者。我们现在可以启动一个消费者，来查看刚才producer产生的数据。请另外打开第四个终端，输入下面命令：

```bash
cd /usr/local/kafka./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic wordsendertest --from-beginning
```

Shell 命令

可以看到，屏幕上会显示出如下结果，也就是刚才你在另外一个终端里面输入的内容：

```
hello hadoop
hello spark
```

到这里，与Kafka相关的准备工作就顺利结束了。注意，现在可以把测试用的第三个和第四个终端关闭掉，第一个终端（正在运行Zookeeper服务）和第二个终端（正在运行Kafka服务）不要关闭，继续留着后面使用。如果记不住是哪个终端，那么所有这些终端窗口都不要关闭，要继续留着后面使用。

## Spark准备工作

Kafka和Flume等高级输入源，需要依赖独立的库（jar文件）。按照我们前面安装好的Spark版本，这些jar包都不在里面，为了证明这一点，我们现在可以测试一下。请打开一个新的终端，然后启动spark-shell：

```bash
cd /usr/local/spark./bin/spark-shell
```

Shell 命令

启动成功后，在spark-shell中执行下面import语句：

```scala
scala> import org.apache.spark.streaming.kafka._<console>:25: error: object kafka is not a member of package org.apache.spark.streaming         import org.apache.spark.streaming.kafka._                                           ^
```

scala

你可以看到，马上会报错，因为找不到相关的jar包。请输入如下命令退出spark-shell：

```scala
scala> :quit
```

scala

根据[Spark官网](http://spark.apache.org/docs/latest/streaming-programming-guide.html)的说明，对于Spark2.1.0版本，如果要使用Kafka，则需要下载spark-streaming-kafka-0-8_2.11相关jar包。
现在请在Linux系统中，打开一个火狐浏览器，[请点击这里访问Spark官网](http://mvnrepository.com/artifact/org.apache.spark/spark-streaming-kafka-0-8_2.11/2.1.0)，里面有提供spark-streaming-kafka-0-8_2.11-2.1.0.jar文件的下载，其中，2.11表示scala的版本，2.1.0表示Spark版本号。下载后的文件会被默认保存在当前Linux登录用户的下载目录下，本教程统一使用hadoop用户名登录Linux系统，所以，文件下载后会被保存到“/home/hadoop/下载”目录下面。现在，我们就把这个文件复制到Spark目录的jars目录下。请新打开一个终端，输入下面命令：

```bash
cd /usr/local/spark/jarsmkdir kafkacd ~cd 下载cp ./spark-streaming-kafka-0-8_2.11-2.1.0.jar /usr/local/spark/jars/kafka
```

Shell 命令

这样，我们就把spark-streaming-kafka-0-8_2.11-2.1.0.jar文件拷贝到了“/usr/local/spark/jars/kafka”目录下。

下面还要继续把Kafka安装目录的libs目录下的所有jar文件复制到“/usr/local/spark/jars/kafka”目录下，请在终端中执行下面命令：

```bash
cd /usr/local/kafka/libslscp ./* /usr/local/spark/jars/kafka
```

Shell 命令

## 编写Spark程序使用Kafka数据源

下面，我们就可以进行程序编写了。请新打开一个终端，然后，执行命令创建代码目录：

```bash
cd /usr/local/spark/mycodemkdir kafkacd kafkamkdir -p src/main/scalacd src/main/scalavim KafkaWordProducer.scala
```

Shell 命令

使用vim编辑器新建了KafkaWordProducer.scala，它是产生一系列字符串的程序，会产生随机的整数序列，每个整数被当做一个单词，提供给KafkaWordCount程序去进行词频统计。请在KafkaWordProducer.scala中输入以下代码：

```
package org.apache.spark.examples.streaming
import java.util.HashMap
import org.apache.kafka.clients.producer.{KafkaProducer, ProducerConfig, ProducerRecord}
import org.apache.spark.SparkConf
import org.apache.spark.streaming._
import org.apache.spark.streaming.kafka._
object KafkaWordProducer {
  def main(args: Array[String]) {
    if (args.length < 4) {
      System.err.println("Usage: KafkaWordCountProducer <metadataBrokerList> <topic> " +
        "<messagesPerSec> <wordsPerMessage>")
      System.exit(1)
    }
    val Array(brokers, topic, messagesPerSec, wordsPerMessage) = args
    // Zookeeper connection properties
    val props = new HashMap[String, Object]()
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, brokers)
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
      "org.apache.kafka.common.serialization.StringSerializer")
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
      "org.apache.kafka.common.serialization.StringSerializer")
    val producer = new KafkaProducer[String, String](props)
   // Send some messages
    while(true) {
      (1 to messagesPerSec.toInt).foreach { messageNum =>
        val str = (1 to wordsPerMessage.toInt).map(x => scala.util.Random.nextInt(10).toString)
          .mkString(" ")
                    print(str)
                    println()
        val message = new ProducerRecord[String, String](topic, null, str)
        producer.send(message)
      }
     Thread.sleep(1000)
    }
  }
}
```

保存后退出vim编辑器。然后，继续在当前目录下创建KafkaWordCount.scala代码文件：

```bash
vim KafkaWordCount.scala
```

Shell 命令

KafkaWordCount.scala是用于单词词频统计，它会把KafkaWordProducer发送过来的单词进行词频统计，代码内容如下：

```
package org.apache.spark.examples.streaming
import org.apache.spark._
import org.apache.spark.SparkConf
import org.apache.spark.streaming._
import org.apache.spark.streaming.kafka._
import org.apache.spark.streaming.StreamingContext._
import org.apache.spark.streaming.kafka.KafkaUtils

object KafkaWordCount{
def main(args:Array[String]){
StreamingExamples.setStreamingLogLevels()
val sc = new SparkConf().setAppName("KafkaWordCount").setMaster("local[2]")
val ssc = new StreamingContext(sc,Seconds(10))
ssc.checkpoint("file:///usr/local/spark/mycode/kafka/checkpoint") //设置检查点，如果存放在HDFS上面，则写成类似ssc.checkpoint("/user/hadoop/checkpoint")这种形式，但是，要启动hadoop
val zkQuorum = "localhost:2181" //Zookeeper服务器地址
val group = "1"  //topic所在的group，可以设置为自己想要的名称，比如不用1，而是val group = "test-consumer-group" 
val topics = "wordsender"  //topics的名称          
val numThreads = 1  //每个topic的分区数
val topicMap =topics.split(",").map((_,numThreads.toInt)).toMap
val lineMap = KafkaUtils.createStream(ssc,zkQuorum,group,topicMap)
val lines = lineMap.map(_._2)
val words = lines.flatMap(_.split(" "))
val pair = words.map(x => (x,1))
val wordCounts = pair.reduceByKeyAndWindow(_ + _,_ - _,Minutes(2),Seconds(10),2) //这行代码的含义在下一节的窗口转换操作中会有介绍
wordCounts.print
ssc.start
ssc.awaitTermination
}
}        
```

保存后退出vim编辑器。然后，继续在当前目录下创建StreamingExamples.scala代码文件：

```bash
vim StreamingExamples.scala
```

Shell 命令

下面是StreamingExamples.scala的代码，用于设置log4j:

```
package org.apache.spark.examples.streaming
import org.apache.spark.internal.Logging
import org.apache.log4j.{Level, Logger}
/** Utility functions for Spark Streaming examples. */
object StreamingExamples extends Logging {
  /** Set reasonable logging levels for streaming if the user has not configured log4j. */
  def setStreamingLogLevels() {
    val log4jInitialized = Logger.getRootLogger.getAllAppenders.hasMoreElements
    if (!log4jInitialized) {
      // We first log something to initialize Spark's default logging, then we override the
      // logging level.
      logInfo("Setting log level to [WARN] for streaming example." +
        " To override add a custom log4j.properties to the classpath.")
      Logger.getRootLogger.setLevel(Level.WARN)
    }
  }
}
```

这样，我们在“/usr/local/spark/mycode/kafka/src/main/scala”目录下，就有了如下三个代码文件：

```
KafkaWordProducer.scala
KafkaWordCount.scala
StreamingExamples.scala
```

然后，请执行下面命令：

```bash
cd /usr/local/spark/mycode/kafka/vim simple.sbt
```

Shell 命令

在simple.sbt中输入以下代码：

```
name := "Simple Project"
version := "1.0"
scalaVersion := "2.11.8"
libraryDependencies += "org.apache.spark" %% "spark-core" % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-streaming_2.11" % "2.1.0"
libraryDependencies += "org.apache.spark" % "spark-streaming-kafka-0-8_2.11" % "2.1.0"
```

保存文件退出vim编辑器。然后执行下面命令，进行打包编译：

```bash
cd /usr/local/spark/mycode/kafka//usr/local/sbt/sbt package
```

Shell 命令

打包成功后，就可以执行程序测试效果了。
首先，如果有用到HDFS，那么请启动hadoop（有可能你前面采用了ssc.checkpoint(“/user/hadoop/checkpoint”)这种形式，写入HDFS）：

```bash
cd /usr/local/hadoop./sbin/start-all.sh
```

Shell 命令

启动hadoop成功以后，就可以测试我们刚才生成的词频统计程序了。
要注意，我们之前已经启动了zookeeper服务，启动了kafka服务，因为我们之前那些终端窗口都没有关闭，所以，这些服务都在运行。如果你不小心关闭了之前的终端窗口，那就请回到本文前面，再启动zookeeper服务，启动kafka服务。

首先，请新打开一个终端，执行如下命令，运行“KafkaWordProducer”程序，生成一些单词（是一堆整数形式的单词）：

```bash
cd /usr/local/spark/usr/local/spark/bin/spark-submit --driver-class-path /usr/local/spark/jars/*:/usr/local/spark/jars/kafka/* --class "org.apache.spark.examples.streaming.KafkaWordProducer" /usr/local/spark/mycode/kafka/target/scala-2.11/simple-project_2.11-1.0.jar localhost:9092 wordsender 3 5
```

Shell 命令

在上面命令中，我们用”–driver-class-path /usr/local/spark/jars/*:/usr/local/spark/jars/kafka/*“来指定应用程序依赖的相关jar包的路径。
注意，上面命令中，”localhost:9092 wordsender 3 5″是提供给KafkaWordProducer程序的4个输入参数，第1个参数localhost:9092是Kafka的broker的地址，第2个参数wordsender是topic的名称，我们在KafkaWordCount.scala代码中已经把topic名称写死掉，所以，KafkaWordCount程序只能接收名称为”wordsender”的topic。第3个参数“3”表示每秒发送3条消息，第4个参数“5”表示，每条消息包含5个单词（实际上就是5个整数）。
执行上面命令后，屏幕上会不断滚动出现新的单词，如下：

```
7 5 0 7 3
2 8 2 1 3
0 1 2 9 2
8 0 9 0 9
9 0 0 6 8
6 6 1 6 5
8 3 6 7 7
3 3 2 6 8
4 5 8 1 5
3 8 8 4 8
2 7 6 3 6
5 7 0 3 6
8 2 9 4 8
2 6 7 6 7
8 8 9 4 5
3 3 2 6 7
0 1 5 8 4
6 1 1 9 0
9 5 6 6 6
2 4 4 2 9
2 0 1 8 8
3 8 4 2 5
```

这个终端窗口就放在这里，不要关闭，千万不要关闭，就让它一直不断发送单词。
然后，请新打开一个终端，执行下面命令，运行KafkaWordCount程序，执行词频统计：

```bash
cd /usr/local/spark/usr/local/spark/bin/spark-submit --driver-class-path /usr/local/spark/jars/*:/usr/local/spark/jars/kafka/* --class "org.apache.spark.examples.streaming.KafkaWordCount" /usr/local/spark/mycode/kafka/target/scala-2.11/simple-project_2.11-1.0.jar
```

Shell 命令

在上面命令中，我们用”–driver-class-path /usr/local/spark/jars/*:/usr/local/spark/jars/kafka/*“来指定应用程序依赖的相关jar包的路径。
运行上面命令以后，就启动了词频统计功能，屏幕上就会显示如下类似信息：

```
-------------------------------------------
Time: 1488156500000 ms
-------------------------------------------
(4,5)
(8,12)
(6,14)
(0,19)
(2,11)
(7,20)
(5,10)
(9,9)
(3,9)
(1,11)
-------------------------------------------
Time: 1488156510000 ms
-------------------------------------------
(4,18)
(8,24)
(6,21)
(0,39)
(2,31)
(7,33)
(5,27)
(9,27)
(3,21)
(1,29)
```