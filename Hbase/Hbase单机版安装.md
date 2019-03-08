下载


JDK下载：

 

Hadoop下载：

http://mirrors.advancedhosters.com/apache/hadoop/common/hadoop-2.6.4/

 

Hbase下载：

http://apache.claz.org/hbase/1.2.6/





相关配置

**一、JDK配置(如已经配了，可不用)**

解压：

[root@centos0 java]# tar zxvfjdk-7u10-linux-i586.tar.gz

配置环境变量：

[root@centos0 java]# vi /etc/profile

 \# set environment value
export JAVA_HOME=/usr/java/jdk1.8
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export HADOOP_HOME=/home/taima/software-package/hadoop-2.6.4
export SPARK_HOME=/home/taima/software-package/spark-1.6.0-bin-hadoop2.6
export HBASE_HOME=/home/taima/software-package/hbase-1.2.6
export HBASE_CONF_DIR=$HBASE_HOME/conf
export HBASE_CLASS_PATH=$HBASE_CONF_DIR

\#path
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$SPARK_HOME/bin:$HBASE_HOME/bin:$PATH





**二、Hbase配置**

1、解压
[root@centos0 bigdata]# tar -zxvf hbase-1.2.6-bin.tar.gz
[root@centos0 bigdata]# mv hbase-1.2.6 hbase 

2、配置hbase-env.sh
[root@centos0 bigdata]# cd /usr/software/bigdata/hbase/conf
[root@centos0 conf]# vi hbase-env.sh
编辑JAVA_HOME环境变量，改变路径到当前JAVA_HOME变量：

export JAVA_HOME=/usr/software/java/jdk1.8
export HBASE_MANAGES_ZK=true



说明：
Hbase依赖于zookeeper，所有的节点和客户端都必须能够访问zookeeper。
**HBase的安装包里面有自带的ZooKeeper，HBASE_MANAGES_ZK环境变量用来设置是使用HBase默认自带的 Zookeeper还是使用独立的ZooKeeper。**
**•	HBASE_MANAGES_ZK为 false 时使用独立的.**
**•	HBASE_MANAGES_ZK为 true 时表示使用默认自带的，让Hbase启动的时候同时也启动自带的ZooKeeper。**

**3、配置hbase-site.xml**
这是HBase的主配置文件。在hbase-site.xml文件里面，找到 <configuration> 和 </configuration> 标签。并在其中，设置属性键名为“hbase.rootdir”。 设置数据保存的目录：

**<font color="red">node1为本机域名，分别在/etc/hosts 和/etc/sysconfig/network 中配置</font>。**



```xml
<configuration>
	<property>
		 <name>hbase.rootdir</name>
		<value>hdfs://node1:8020/hbase</value>
        </property>
	<property>
    		<name>hbase.cluster.distributed</name>
    		<value>true</value>
  	</property>
	<property>
                <name>hbase.master</name>
                <value>16000</value>
        </property>
	<property>
                <name>hbase.regionserver.port</name>
                <value>16020</value>
        </property>

	<property>
        	<name>hbase.zookeeper.property.dataDir</name>
          	<value>/home/taima/software-package/zookeeper-data</value>                                                                                    
  	</property>
	<property>
		<name>zookeeper.znode.parent</name>
		<value>/hbase</value>
	</property>
	<property>
                <name>hbase.zookeeper.quorum</name>
                <value>node1</value>
        </property>
        <property>
                <name>hbase.client.retries.number</name>
                <value>35</value>
        </property>
        <property>
                <name>hbase.master.hostname</name>
                <value>node1</value>
        </property>
        <property>
                <name>hbase.regionserver.hostname</name>
                <value>node1</value>
        </property>
</configuration>
```

**4、启动hbase（先启动zookeeper，再启动hbase）**

**去hbase路径下面**

[root@centos0 ~]# cd /usr/software/bigdata/hbase/bin
[root@centos0 bin]# ./start-hbase.sh
starting master, logging to /usr/software/bigdata/hbase/logs/hbase-root-master-centos0.out

启动成功后，可以通过命令查看当前的Hbase版本 
[root@centos0 ~]# hbase version
HBase 1.2.6
Source code repository file:///home/busbey/projects/hbase/hbase-assembly/target/hbase-1.2.6 revision=Unknown
Compiled by busbey on Mon May 29 02:25:32 CDT 2017
From source with checksum 7e8ce83a648e252758e9dae1fbe779c9

查看正在运行的
[root@centos0 bin]# jps
1730 Jps
1335 HMaster
HMaster  (由于是单机模式，所以只有HMaster在运行)

可以输入命令进入Hbase, 使用 "hbase shell" 命令可以连接到正在运行的 HBase 实例.
[root@centos0 ~]#  hbase shell
hbase(main):001:0>

至此单机版Hbase配置完成，浏览器访问http://ipxxxxxxxxx:16010



**5，zookeeper配置**

配置conf/zoo.cfg文件：

```
clientPort=2181
server.1=node1:2888:3888
```

并也拷贝一份zoo.cfg到hbase/conf 路径下面.









#### **hbase-site.xml** **配置参数解析**

- **hbase.rootdir**

这个目录是 RegionServer 的共享目录，用来持久化 HBase。特别注意的是 hbase.rootdir 里面的 HDFS 地址是要跟 Hadoop 的 core-site.xml 里面的 fs.defaultFS 的 HDFS 的 IP 地址或者域名、端口必须一致。（HA环境下，dfs.nameservices  是由zookeeper来决定的）

- **hbase.cluster.distributed**

HBase 的运行模式。为 false 表示单机模式，为 true 表示分布式模式。若为 false，HBase 和 ZooKeeper 会运行在同一个 JVM 中

- **hbase.master**

如果只设置单个 Hmaster，那么 hbase.master 属性参数需要设置为 master:16000 (主机名:16000)

如果要设置多个 Hmaster，那么我们只需要提供端口 16000，因为选择真正的 master 的事情会有 zookeeper 去处理

- **hbase.tmp.dir**

本地文件系统的临时文件夹。可以修改到一个更为持久的目录上。(/tmp会在重启时清除)

- **hbase.zookeeper.quorum**

对于 ZooKeeper 的配置。至少要在 hbase.zookeeper.quorum 参数中列出全部的 ZooKeeper 的主机，用逗号隔开。该属性值的默认值为 localhost，这个值显然不能用于分布式应用中。

- **hbase.zookeeper.property.dataDir**

这个参数用户设置 ZooKeeper 快照的存储位置，默认值为 /tmp，显然在重启的时候会清空。因为笔者的 ZooKeeper 是独立安装的，所以这里路径是指向了 $ZOOKEEPER_HOME/conf/zoo.cfg 中 dataDir 所设定的位置。

- **hbase.zookeeper.property.clientPort**

表示客户端连接 ZooKeeper 的端口。

- **zookeeper.session.timeout**

ZooKeeper 会话超时。Hbase 把这个值传递改 zk 集群，向它推荐一个会话的最大超时时间

- **hbase.regionserver.restart.on.zk.expire**

当 regionserver 遇到 ZooKeeper session expired ， regionserver 将选择 restart 而不是 abort。



Hbase单机版默认版本是16010 ，可以看到Hbase视图界面
--------------------- 
