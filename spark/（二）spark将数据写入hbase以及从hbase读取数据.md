# spark将数据写入hbase以及从hbase读取数据 

1、[Spark](http://lib.csdn.net/base/spark)如何利用saveAsHadoopDataset和saveAsNewAPIHadoopDataset将RDD写入[Hbase](http://lib.csdn.net/base/hbase)

2、spark从[hbase](http://lib.csdn.net/base/hbase)中读取数据并转化为RDD

操作方式为在eclipse本地运行spark连接到远程的hbase。

[Java](http://lib.csdn.net/base/java)版本：1.7.0

[Scala](http://lib.csdn.net/base/scala)版本：2.10.4

zookeeper版本：3.4.5（禁用了hbase自带zookeeper，选择自己部署的）

[Hadoop](http://lib.csdn.net/base/hadoop)版本：2.4.1

spark版本：1.6.1

hbase版本：1.2.3

集群：centos6.5_x64

# 将RDD写入hbase

**注意点：**

依赖：

将lib目录下的[hadoop](http://lib.csdn.net/base/hadoop)开头jar包、hbase开头jar包添加至classpath

此外还有lib目录下的：zookeeper-3.4.6.jar、metrics-core-2.2.0.jar（缺少会提示hbase  RpcRetryingCaller: Call  exception不断尝试重连hbase，不报错）、htrace-core-3.1.0-incubating.jar、guava-12.0.1.jar

$SPARK_HOME/lib目录下的 spark-assembly-1.6.1-hadoop2.4.0.jar

不同的package中可能会有相同名称的类，不要导错

连接集群：

spark应用需要连接到zookeeper集群，然后借助zookeeper访问hbase。一般可以通过两种方式连接到zookeeper：

第一种是将hbase-site.xml文件加入classpath

第二种是在HBaseConfiguration实例中设置

如果不设置，默认连接的是localhost:2181会报错：connection refused 

本文使用的是第二种方式。

hbase创建表：

虽然可以在spark应用中创建hbase表，但是不建议这样做，最好在hbase shell中创建表，spark写或读数据

**使用saveAsHadoopDataset写入数据**





```scala
package com.test

import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.mapred.TableOutputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.mapred.JobConf
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD.rddToPairRDDFunctions

object TestHBase {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("HBaseTest").setMaster("local")
    val sc = new SparkContext(sparkConf)

    val conf = HBaseConfiguration.create()
    //设置zooKeeper集群地址，也可以通过将hbase-site.xml导入classpath，但是建议在程序里这样设置
    conf.set("hbase.zookeeper.quorum","slave1,slave2,slave3")
    //设置zookeeper连接端口，默认2181
    conf.set("hbase.zookeeper.property.clientPort", "2181")

    val tablename = "account"
    
    //初始化jobconf，TableOutputFormat必须是org.apache.hadoop.hbase.mapred包下的！
    val jobConf = new JobConf(conf)
    jobConf.setOutputFormat(classOf[TableOutputFormat])
    jobConf.set(TableOutputFormat.OUTPUT_TABLE, tablename)
    
    val indataRDD = sc.makeRDD(Array("1,jack,15","2,Lily,16","3,mike,16"))


    val rdd = indataRDD.map(_.split(',')).map{arr=>{
      /*一个Put对象就是一行记录，在构造方法中指定主键
       * 所有插入的数据必须用org.apache.hadoop.hbase.util.Bytes.toBytes方法转换
       * Put.add方法接收三个参数：列族，列名，数据
       */
      val put = new Put(Bytes.toBytes(arr(0).toInt))
      put.add(Bytes.toBytes("cf"),Bytes.toBytes("name"),Bytes.toBytes(arr(1)))
      put.add(Bytes.toBytes("cf"),Bytes.toBytes("age"),Bytes.toBytes(arr(2).toInt))
      //转化成RDD[(ImmutableBytesWritable,Put)]类型才能调用saveAsHadoopDataset
      (new ImmutableBytesWritable, put) 
    }}
    
    rdd.saveAsHadoopDataset(jobConf)
    
    sc.stop()
  }

}
```

使用saveAsNewAPIHadoopDataset写入数据



```
package com.test

import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat
import org.apache.spark._
import org.apache.hadoop.mapreduce.Job
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.client.Result
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.util.Bytes

object TestHBase3 {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("HBaseTest").setMaster("local")
    val sc = new SparkContext(sparkConf)
    
    val tablename = "account"
    
    sc.hadoopConfiguration.set("hbase.zookeeper.quorum","slave1,slave2,slave3")
    sc.hadoopConfiguration.set("hbase.zookeeper.property.clientPort", "2181")
    sc.hadoopConfiguration.set(TableOutputFormat.OUTPUT_TABLE, tablename)
    
    val job = new Job(sc.hadoopConfiguration)
    job.setOutputKeyClass(classOf[ImmutableBytesWritable])
    job.setOutputValueClass(classOf[Result])  
    job.setOutputFormatClass(classOf[TableOutputFormat[ImmutableBytesWritable]])  

    val indataRDD = sc.makeRDD(Array("1,jack,15","2,Lily,16","3,mike,16"))
    val rdd = indataRDD.map(_.split(',')).map{arr=>{
      val put = new Put(Bytes.toBytes(arr(0)))
      put.add(Bytes.toBytes("cf"),Bytes.toBytes("name"),Bytes.toBytes(arr(1)))
      put.add(Bytes.toBytes("cf"),Bytes.toBytes("age"),Bytes.toBytes(arr(2).toInt))
      (new ImmutableBytesWritable, put) 
    }}
    
    rdd.saveAsNewAPIHadoopDataset(job.getConfiguration())
  }

}
```



#  从hbase读取数据转化成RDD



本例基于官方提供的例子

```scala
package com.test

import org.apache.hadoop.hbase.{HBaseConfiguration, HTableDescriptor, TableName}
import org.apache.hadoop.hbase.client.HBaseAdmin
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.spark._
import org.apache.hadoop.hbase.client.HTable
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat
import org.apache.hadoop.mapred.JobConf
import org.apache.hadoop.io._

object TestHBase2 {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("HBaseTest").setMaster("local")
    val sc = new SparkContext(sparkConf)
    
	val tablename = "account"
    val conf = HBaseConfiguration.create()
    //设置zooKeeper集群地址，也可以通过将hbase-site.xml导入classpath，但是建议在程序里这样设置
    conf.set("hbase.zookeeper.quorum","slave1,slave2,slave3")
    //设置zookeeper连接端口，默认2181
    conf.set("hbase.zookeeper.property.clientPort", "2181")
    conf.set(TableInputFormat.INPUT_TABLE, tablename)

    // 如果表不存在则创建表
    val admin = new HBaseAdmin(conf)
    if (!admin.isTableAvailable(tablename)) {
      val tableDesc = new HTableDescriptor(TableName.valueOf(tablename))
      admin.createTable(tableDesc)
    }

    //读取数据并转化成rdd
    val hBaseRDD = sc.newAPIHadoopRDD(conf, classOf[TableInputFormat],
      classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable],
      classOf[org.apache.hadoop.hbase.client.Result])

    val count = hBaseRDD.count()
    println(count)
    hBaseRDD.foreach{case (_,result) =>{
      //获取行键
      val key = Bytes.toString(result.getRow)
      //通过列族和列名获取列
      val name = Bytes.toString(result.getValue("cf".getBytes,"name".getBytes))
      val age = Bytes.toInt(result.getValue("cf".getBytes,"age".getBytes))
      println("Row key:"+key+" Name:"+name+" Age:"+age)
    }}

    sc.stop()
    admin.close()
  }
}
```