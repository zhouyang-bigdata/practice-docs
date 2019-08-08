## **Hadoop2.7.2之集群搭建（单机）**

#### <font color="red">cloudera5.13适配的是2.6.4(以上)</font>。

1、下载地址

```
http://hadoop.apache.org/releases.html

```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519112332287)



我下载的是2.7.2，官网在2.5之后默认提供的就是64位的，这里直接下载下来用即可

2、安装[Hadoop](http://lib.csdn.net/base/hadoop)

```
tar -zxvf hadoop-2.7.2.tar.gz -C /opt/soft

```

3、查看Hadoop是32 or 64 位
 参考：<http://www.aboutyun.com/thread-12796-1-1.html>

```
cd /opt/soft/hadoop-2.7.2/lib/native
file libhadoop.so.1.0.012

```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519111342189)

4、配置/etc/hosts

#### <font color="red">node1.myexample.com为本机的域名</font>



```
vi /etc/hosts

```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160526210846523)



```shell
vi /etc/sysconfig/network


```

修改hostname为本机域名。



### 配置环境变量

```shell
vi /etc/profile
```



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







## 配置启动Hadoop**

1、修改hadoop2.7.2/etc/hadoop/hadoop-env.sh指定JAVA_HOME

```
# The java implementation to use.
export JAVA_HOME=/opt/soft/jdk1.8.0_91

```

2、修改hdfs的配置文件

修改hadoop2.7.2/etc/hadoop/core-site.xml 如下：

```
<configuration>
   		<property>
                <name>fs.default.name</name>
                <value>hdfs://node1.myexample.com:8020</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
        		<value>file:/home/taima/software-package/hadoop-2.6.4/dfs</value>
                <description>Abase for other temporary directories.</description>
        </property>
</configuration>
```

这里fs.defaultFS的value最好是写本机的静态IP当然写本机主机名，再配置hosts是最好的，如果用localhost，然后在windows用java操作hdfs的时候，会连接不上主机。

修改hadoop2.7.2/etc/hadoop/hdfs-site.xml 如下：

```
<configuration>
                <property>
                        <!-- 数据备份数3 -->
               		<name>dfs.replication</name>
                        <value>1</value>
                </property>
                <property>
                    	<!-- nameNode 节点的本地元数据备份目录-->
                	<name>dfs.name.dir</name>
                        <value>file:/home/taima/software-package/hadoop-2.6.4/dfs/dfs/name</value>
                </property>
                <property>
                        <name>dfs.data.dir</name>
                        <value>file:/home/taima/software-package/hadoop-2.6.4/dfs/dfs/data</value>
                </property>
		<property>
		        <!-- nameNode 默认256-->
                        <name>dfs.datanode.max.xcievers</name>
                        <value>4096</value>
               </property>
               <property>
               		<!-- 默认10-->
               		<name>dfs.namenode.handler.count</name>
               		<value>30</value>
	       </property>
	       <property>
	               <name>dfs.client.block.write.replace-datanode-on-failure.enable</name>        
	       	       <value>true</value>
	       </property>
	       <property>
	           <name>dfs.client.block.write.replace-datanode-on-failure.policy</name>
	           <value>NEVER</value>
	       </property>
	       <property>
	                               <!-- 默认10-->
                        <name>dfs.datanode.handler.count</name>
                        <value>30</value>
               </property>
</configuration>
```

3、配置SSH免密码登录

配置前：

```
ssh localhost

```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519114111356)

会出现如上效果，要求我输入本机登录密码

配置方法：

```
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

配置后，不用密码可以直接登录了

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519114242920)

4、hdfs启动与停止

第一次启动得先格式化（最好不要复制）：

```
./bin/hdfs namenode -format
```

启动hdfs

```
./sbin/start-dfs.sh
```

看到如下效果表示成功：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519124508945)

[测试](http://lib.csdn.net/base/softwaretest)用浏览器访问50070：（如果没响应，则在防火墙白名单中加入50070端口）

```shell
firewall-cmd --zone=public --add-port=50070/tcp --permanent
firewall-cmd --reload


```

http://192.168.2.100:50070/

效果如下：
![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519124621133)

停止hdfs

```
sbin/stop-dfs.sh
```

5、常用操作：
HDFS shell
查看帮助

```
hadoop fs -help <cmd>
```

上传

```
hadoop fs -put <linux上文件>  <hdfs上的路径>
```

查看文件内容

```
hadoop fs -cat <hdfs上的路径>
```

查看文件列表

```
hadoop fs -ls /
```

下载文件

```
hadoop fs -get <hdfs上的路径>  <linux上文件>
```

**上传文件测试**
 创建一个words.txt 文件并上传

```
vi words.txt

Hello World
Hello Tom
Hello Jack
Hello Hadoop
Bye   hadoop
```

将words.txt上传到hdfs的根目录

```
bin/hadoop fs -put words.txt /
```

可以通过浏览器访问：<http://192.168.2.100:50070/>

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519132722448)

这里的words.txt就是我们上传的words.txt

## **配置启动YARN**

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519125030714)

从上图看看出我们的MapReduce是运行在YARN上的，而YARN是运行在HDFS之上的，我们已经安装了HDFS现在来配置启动YARN，然后运行一个WordCount程序。

1、配置etc/hadoop/mapred-site.xml：

mv  mapred-site.xml.template mapred-site.xml

```
<configuration>
    <!-- 通知框架MR使用YARN -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

```

2、配置etc/hadoop/yarn-site.xml:

```
<configuration>
	<property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>

```

3、YARN的启动与停止

启动

```
./sbin/start-yarn.sh
```

如下：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519130144945)

测试用浏览器访问8088：（如果没响应，则在防火墙白名单加入8088端口）



![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160526221123767)

停止

```
sbin/stop-yarn.sh
```

现在我们的hdfs和yarn都运行成功了，我们开始运行一个WordCount的MP程序来测试我们的单机模式集群是否可以正常工作。

## **运行一个简单的MP程序**

我们的MapperReduce将会跑在YARN上，结果将存在HDFS上：

```
./bin/hadoop jar /opt/soft/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar  wordcount hdfs://localhost:9000/words.txt hdfs://localhost:9000/out
```

用hadoop执行一个叫 hadoop-mapreduce-examples.jar 的 wordcount 方法，其中输入参数为 hdfs上根目录的words.txt 文件，而输出路径为 hdfs跟目录下的out目录，运行过程如下：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519134620458)

我们通过浏览器访问和下载查看结果：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160519135623866)

这里下载的时候会跳转到另一个地址如下：

```
http://singlenode:50075/webhdfs/v1/out/part-r-00000?op=OPEN&namenoderpcaddress=localhost:9000&offset=0
```

1、需把singlenode换成192.168.2.100或是在hosts里加入 192.168.2.100 singlenode 隐射关系

2、需开放50075端口。

下载下来结果如下：

```
Bye 1
Hadoop  2
Hello   4
Jack    1
Tom 1
World   1
```

说明我们已经计算出了，单词出现的次数。

至此，我们Hadoop的单机模式搭建成功。