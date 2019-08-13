# impala2.12 详细安装教程



# 前言

如果你认为impala的安装真的只是安装impala那么就大错特错了。因为impala依赖于hive存储元数据，所以你需要安装hive,因为hive依赖hadoop，所以你需要安装hadoop，因为hive的元数据存在关系数据库中比如mysql，所以你需要安装mysql。又因为impala需要依赖sentry的jar包，所以你还需要下载sentry。最令人发指的是如果你不想用cloudera manager安装impala，就需要知己确保以上各个组件的版本兼容，稍有差错就会各种问题。经过各种坑，献上我的安装步骤。由于我的环境hadoop已经装好了，所以本文不会涉及hadoop安装，大家可以自己去找。另外本文的安装方式是rpm包安装的，如果看官想知道如何自己编译源码，安装部署，可以看[impala2.12.0的编译与安装](https://blog.csdn.net/qqqq0199181/article/details/98515118)

# 安装mysql 安装

## 安装包

MySQL-client-5.6.41-1.el6.x86_64.rpm
MySQL-server-5.6.41-1.el6.x86_64.rpm

## 移除旧包

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181210202224158.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181210202233706.png)

rm -rf /etc/my.cnf.*
rm -rf /etc/my.cnf.*

## 安装rpm包

rpm -ivh MySQL-server-5.6.41-1.el6.x86_64.rpm

rpm -ivh MySQL-client-5.6.41-1.el6.x86_64.rpm

查看mysql初始登录密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181210202300672.png)

## 启动 mysql服务

service mysql start

## 修改登录密码

mysql –uroot –p初始密码

mysql >set PASSWORD=PASSWORD(‘密码’)；

## 赋予远程登录权限

mysql> GRANT ALL PRIVILEGES ON *.* TO ‘root’@’%’ IDENTIFIED BY ‘root’ WITH GRANT OPTION;

mysql> GRANT ALL PRIVILEGES ON *.* TO ‘root’@‘hadoop02’ IDENTIFIED BY ‘root’ WITH GRANT OPTION;

mysql> GRANT ALL PRIVILEGES ON *.* TO ‘root’@’%’ IDENTIFIED BY ‘root’ WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;

# 安装hive1.1.0

## 安装包

apache-hive-1.1.0-bin.tar

## 解压

tar -zxvf apache-hive-1.1.0-bin.tar.gz

## 配置环境变量

编辑/etc/profile，添加

```
export HADOOP_HOME=/usr/lib/cluster001/SERVICE-HADOOP-97a10ac9a6f044fd8e844b9f6afce095
export HIVE_HOME=/usr/lib/hive/apache-hive-1.1.0-bin
export PATH=$PATH:$HADOOP_HOME/bin:$HIVE_HOME/bin

```

执行source /etc/profile

## 验证hive

执行hive –version

输出如下：

```
Hive 1.1.0
Subversion git://localhost.localdomain/Users/noland/workspaces/hive-apache/hive -r 3b87e226d9f2ff5d69385ed20704302cffefab21
Compiled by noland on Wed Feb 18 16:06:08 PST 2015
From source with checksum bca57a923a7578b7e5e9350ffb165cca
1234
```

## 配置

在hive 的config目录下创建hive-site.xml，添加如下内容

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value> mysql数据库密码
    </property>
   <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://127.0.0.1:3306/hive</value> url
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value> 驱动
    </property>

</configuration>

```

## 复制mysql的驱动程序到hive/lib下面

cp mysql-connector-java-5.1.47-bin.jar lib/

## 在mysql中创建数据库hive

mysql> create database hive;
Query OK, 1 row affected (0.00 sec)

## 初始化schema

进入hive的bin目录下
schematool -dbType mysql –initSchema

如果成功输出如下：

```
Metastore connection URL:	 jdbc:mysql://127.0.0.1:3306/hive
Metastore Connection Driver :	 com.mysql.jdbc.Driver
Metastore connection User:	 root
Starting metastore schema initialization to 1.1.0
Initialization script hive-schema-1.1.0.mysql.sql
Initialization script completed
schemaTool completed
```

## 进入hive

[root@fff apache-hive-1.1.0-bin]# hive

Logging initialized using configuration in jar:file:/usr/lib/hive/apache-hive-1.1.0-bin/lib/hive-common-1.1.0.jar!/hive-log4j.properties
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/lib/cluster001/SERVICE-HADOOP-97a10ac9a6f044fd8e844b9f6afce095/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/lib/hive/apache-hive-1.1.0-bin/lib/hive-jdbc-1.1.0-standalone.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See <http://www.slf4j.org/codes.html#multiple_bindings> for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
hive> quit;

## hive安装完成

# 安装sentry1.5.1

## 安装包

sentry-1.5.1+cdh5.14.2+444-1.cdh5.14.2.p0.11.el6.noarch.rpm

## 安装

rpm -ivh --nodeps sentry-1.5.1+cdh5.14.2+444-1.cdh5.14.2.p0.11.el6.noarch.rpm

# 安装impala2.12.0

## 安装包

bigtop-utils-0.7.0+cdh5.3.3+0-1.cdh5.3.3.p0.8.el6.noarch.rpm
impala-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-catalog-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-debuginfo-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-server-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-shell-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-state-store-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
impala-udf-devel-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
**注意：** 如果是hadoop集群的话，impala-server impala-shell 需要安装在每个datanode所在节点上，因为impala是MPP结构，需要通过分布在各个节点上的server去并行处理请求的。

## 安装rpm包

rpm -ivh --nodeps impala-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-catalog-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-debuginfo-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-server-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-shell-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-state-store-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm
rpm -ivh --nodeps impala-udf-devel-2.12.0+cdh5.15.1+0-1.cdh5.15.1.p0.4.el7.x86_64.rpm

## 重新创建包软连接

使用rpm安装之后在impala/lib下面会发现很多无效软连接，全部删除
cd /usr/lib/impala/lib
sudo rm -rf avro*.jar
sudo rm -rf hadoop-*.jarsudo rm -rf hive-*.jar
sudo rm -rf hbase-*.jarsudo rm -rf parquet-hadoop-bundle.jarsudo rm -rf sentry-*.jar
sudo rm -rf zookeeper.jar
sudo rm -rf [libhadoop.so](http://libhadoop.so/)
sudo rm -rf libhadoop.so.1.0.0
sudo rm -rf [libhdfs.so](http://libhdfs.so/)

sudo rm -rf libhdfs.so.0.0.0

删除之后：重新创建软连接：脚本如下：

注意事项：脚本中jar包版本是自己环境中的jar包版本，不要直接运用，必须修改，否则依然找不到软连接

```
HBASE_HOME=/usr/lib/hbase
HADOOP_HOME=/usr/lib/hadoop
HIVE_HOME=/usr/lib/hive
ZK_HOME=/usr/lib/zookeeper
SENTRY_HOME=/usr/lib/sentry

sudo ln -s $HADOOP_HOME/share/hadoop/common/lib/hadoop-annotations-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-annotations.jar
sudo ln -s $HADOOP_HOME/share/hadoop/common/lib/hadoop-auth-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-auth.jar
sudo ln -s $HADOOP_HOME/share/hadoop/tools/lib/hadoop-aws-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-aws.jar
sudo ln -s $HADOOP_HOME/share/hadoop/common/hadoop-common-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-common.jar
sudo ln -s $HADOOP_HOME/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-hdfs.jar
sudo ln -s $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-mapreduce-client-common.jar
sudo ln -s $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-mapreduce-client-core.jar
sudo ln -s $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-mapreduce-client-jobclient.jar
sudo ln -s $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-mapreduce-client-shuffle.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-api-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-api.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-client-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-client.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-common-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-common.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-server-applicationhistoryservice.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-server-common-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-server-common.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-server-nodemanager.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-server-resourcemanager.jar
sudo ln -s $HADOOP_HOME/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.6.0-cdh5.7.0.jar /usr/lib/impala/lib/hadoop-yarn-server-web-proxy.jar
sudo ln -s $HBASE_HOME/lib/avro-1.7.6-cdh5.7.0.jar /usr/lib/impala/lib/avro.jar
sudo ln -s $HBASE_HOME/lib/hbase-annotations-1.2.0-cdh5.7.0.jar /usr/lib/impala/lib/hbase-annotations.jar
sudo ln -s $HBASE_HOME/lib/hbase-client-1.2.0-cdh5.7.0.jar /usr/lib/impala/lib/hbase-client.jar
sudo ln -s $HBASE_HOME/lib/hbase-common-1.2.0-cdh5.7.0.jar /usr/lib/impala/lib/hbase-common.jar
sudo ln -s $HBASE_HOME/lib/hbase-protocol-1.2.0-cdh5.7.0.jar /usr/lib/impala/lib/hbase-protocol.jar
sudo ln -s $HIVE_HOME/lib/ant-1.9.1.jar /usr/lib/impala/lib/hive-ant.jar
sudo ln -s $HIVE_HOME/lib/hive-beeline-1.1.0.jar /usr/lib/impala/lib/hive-beeline.jar
sudo ln -s $HIVE_HOME/lib/hive-common-1.1.0.jar /usr/lib/impala/lib/hive-common.jar
sudo ln -s $HIVE_HOME/lib/hive-exec-1.1.0.jar /usr/lib/impala/lib/hive-exec.jar
sudo ln -s $HIVE_HOME/lib/hive-hbase-handler-1.1.0.jar /usr/lib/impala/lib/hive-hbase-handler.jar
sudo ln -s $HIVE_HOME/lib/hive-metastore-1.1.0.jar /usr/lib/impala/lib/hive-metastore.jar
sudo ln -s $HIVE_HOME/lib/hive-serde-1.1.0.jar /usr/lib/impala/lib/hive-serde.jar
sudo ln -s $HIVE_HOME/lib/hive-service-1.1.0.jar /usr/lib/impala/lib/hive-service.jar
sudo ln -s $HIVE_HOME/lib/hive-shims-common-1.1.0.jar /usr/lib/impala/lib/hive-shims-common.jar
sudo ln -s $HIVE_HOME/lib/hive-shims-1.1.0.jar /usr/lib/impala/lib/hive-shims.jar
sudo ln -s $HIVE_HOME/lib/hive-shims-scheduler-1.1.0.jar /usr/lib/impala/lib/hive-shims-scheduler.jar
sudo ln -s $HADOOP_HOME/lib/native/libhadoop.so /usr/lib/impala/lib/libhadoop.so
sudo ln -s $HADOOP_HOME/lib/native/libhadoop.so.1.0.0 /usr/lib/impala/lib/libhadoop.so.1.0.0
sudo ln -s $HADOOP_HOME/lib/native/libhdfs.so /usr/lib/impala/lib/libhdfs.so
sudo ln -s $HADOOP_HOME/lib/native/libhdfs.so.0.0.0 /usr/lib/impala/lib/libhdfs.so.0.0.0
sudo ln -s $HIVE_HOME/lib/parquet-hadoop-bundle-1.6.0rc3.jar /usr/lib/impala/lib/parquet-hadoop-bundle.jar
sudo ln -s $SENTRY_HOME/lib/sentry-binding-hive-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-binding-hive.jar
sudo ln -s $SENTRY_HOME/lib/sentry-core-common-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-core-common.jar
sudo ln -s $SENTRY_HOME/lib/sentry-core-model-db-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-core-model-db.jar
sudo ln -s $SENTRY_HOME/lib/sentry-policy-common-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-policy-common.jar
sudo ln -s $SENTRY_HOME/lib/sentry-policy-db-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-policy-db.jar
sudo ln -s $SENTRY_HOME/lib/sentry-provider-cache-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-provider-cache.jar
sudo ln -s $SENTRY_HOME/lib/sentry-provider-common-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-provider-common.jar
sudo ln -s $SENTRY_HOME/lib/sentry-provider-db-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-provider-db.jar
sudo ln -s $SENTRY_HOME/lib/sentry-provider-file-1.5.1-cdh5.14.2.jar /usr/lib/impala/lib/sentry-provider-file.jar

```

## 安装bigtop-utils

rpm -ivh bigtop-utils-0.7.0+cdh5.3.3+0-1.cdh5.3.3.p0.8.el6.noarch.rpm

## 修改bigtop-utils配置

cd /etc/default/

vim bigtop-utils

添加：export JAVA_HOME=/soft/java/jdk1.8.0_65 //对应自己的java_home

Source /etc/default/bigtop-utils

## 复制xml文件到impala配置路径

cp $HADOOP_HOME/etc/hadoop/hdfs-site.xml /etc/impala/conf

修改hdfs-site.xml

```
<property>
    <name>dfs.client.read.shortcircuit</name>
    <value>true</value>
</property>
<property>
    <name>dfs.domain.socket.path</name>
    <value>/var/run/hadoop-hdfs/dn</value>
</property>
<property>
  <name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
  <value>true</value>
</property>
<property>
   <name>dfs.client.use.legacy.blockreader.local</name>
   <value>false</value>
</property>
<property>
   <name>dfs.datanode.data.dir.perm</name>
   <value>750</value>
</property>
<property>
   <name>dfs.block.local-path-access.user</name>
   <value>hadoop</value>
</property>
<property>
   <name>dfs.client.file-block-storage-locations.timeout</name>
   <value>3000</value>
</property>

```

cp $HADOOP_HOME/etc/hadoop/core-site.xml /etc/impala/conf
修改core-site配置

```
<property>
　　　　　　<name>dfs.client.read.shortcircuit</name>
　　　　　　　　<value>true</value>
　　　　　　</property>
　　　　　　<property>
　　　　　　　　<name>dfs.client.read.shortcircuit.skip.checksum</name>
　　　　　　　　<value>false</value>
　　　　　　</property>
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop主节点ip:8020</value>
            </property>
　　　　　　<property>
　　　　　　　　<name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
　　　　　　　　<value>true</value>
　　　　　　</property>

<property>
　　　　　　　　<name>fs.AbstractFileSystem.adl.impl</name>
　　　　　　　　<value>org.apache.hadoop.fs.adl.Adl</value>
　　　　　　</property>
　　　　　　<property>
　　　　　　　　<name>fs.adl.impl</name>
　　　　　　　　<value>org.apache.hadoop.fs.adl.AdlFileSystem</value>
　　　　　　</property>

```

cp $HIVE_HOME/conf/hive-site.xml /etc/impala/conf

## 将hadoop-azure-datalake.jar拷贝到impala的lib下

这步很重要，被坑惨过

## 复制mysql-connector jar包到/usr/share/java并改名

cp /usr/lib/hive/apache-hive-1.1.0-bin/lib/mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar

## 修改配置文件

vi /etc/default/impala
修改后如下：

```
IMPALA_CATALOG_SERVICE_HOST=127.0.0.1
IMPALA_STATE_STORE_HOST=127.0.0.1
IMPALA_STATE_STORE_PORT=24000
IMPALA_BACKEND_PORT=22000
IMPALA_LOG_DIR=/var/log/impala

IMPALA_CATALOG_ARGS=" -log_dir=${IMPALA_LOG_DIR} "
IMPALA_STATE_STORE_ARGS=" -log_dir=${IMPALA_LOG_DIR} -state_store_port=${IMPALA_STATE_STORE_PORT}"
IMPALA_SERVER_ARGS=" \
    -log_dir=${IMPALA_LOG_DIR} \
    -catalog_service_host=${IMPALA_CATALOG_SERVICE_HOST} \
    -state_store_port=${IMPALA_STATE_STORE_PORT} \
    -use_statestore \
    -state_store_host=${IMPALA_STATE_STORE_HOST} \
    -be_port=${IMPALA_BACKEND_PORT} \
    -kudu_master_hosts=128-39:7051 "

ENABLE_CORE_DUMPS=true

```

配置hadoop 的hdfs-size.xml
添加

```
<property>
        <name>dfs.client.read.shortcircuit</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.domain.socket.path</name>
        <value>/var/run/hadoop-hdfs/dn</value>
    </property>
    <property>
          <name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
          <value>true</value>
    </property>
    <property>
           <name>dfs.client.use.legacy.blockreader.local</name>
           <value>false</value>
    </property>
    <property>
       <name>dfs.datanode.data.dir.perm</name>
       <value>750</value>
    </property>
    <property>
       <name>dfs.block.local-path-access.user</name>
       <value>hadoop</value>
    </property>
    <property>
       <name>dfs.client.file-block-storage-locations.timeout</name>
       <value>3000</value>
</property>

```

创建/var/run/hadoop-hdfs 目录
重启hadoop

## 启动impala服务

hive --service metastore &
hive --service hiveserver2 &

service impala-state-store start

service impala-catalog start

service impala-server start

## 启动impala-shell

show databases;//验证操作

# 参考资料

ClouderaManager安装：<https://blog.csdn.net/u011026329/article/details/79166626>
mpala安装教程–全网最详细可靠： <https://blog.csdn.net/m0_38003171/article/details/79851240>