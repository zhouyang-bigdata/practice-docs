# Maven构建Hadoop项目及代码下载

**前言**

Hadoop的MapReduce环境是一个复杂的编程环境，所以我们要尽可能地简化构建MapReduce项目的过程。Maven是一个很不错的自动化项目构建工具，通过Maven来帮助我们从复杂的环境配置中解脱出来，从而标准化开发过程。所以，写MapReduce之前，让我们先花点时间把刀磨快！！当然，除了Maven还有其他的选择Gradle(推荐), Ivy….

后面将会有介绍几篇MapReduce开发的文章，都要依赖于本文中Maven的构建的MapReduce环境。

**目录**

- Maven介绍
- Maven安装(win)
- Hadoop开发环境介绍
- 用Maven构建Hadoop环境
- MapReduce程序开发
- 模板项目上传github

\1. Maven介绍

Apache Maven，是一个Java的项目管理及自动构建工具，由Apache软件基金会所提供。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。曾是Jakarta项目的子项目，现为独立Apache项目。

maven的开发者在他们开发网站上指出，maven的目标是要使得项目的构建更加容易，它把编译、打包、测试、发布等开发过程中的不同环节有机的串联了起来，并产生一致的、高质量的项目信息，使得项目成员能够及时地得到反馈。maven有效地支持了测试优先、持续集成，体现了鼓励沟通，及时反馈的软件开发理念。如果说Ant的复用是建立在”拷贝–粘贴”的基础上的，那么Maven通过插件的机制实现了项目构建逻辑的真正复用。

\2. Maven安装(win)

下载Maven：<http://maven.apache.org/download.cgi>

下载最新的xxx-bin.zip文件，在win上解压到 D:\toolkit\maven3

并把maven/bin目录设置在环境变量PATH：

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/203954x7nzioz7n0j07lyq.png)](http://blog.fens.me/wp-content/uploads/2013/09/win7-maven.png)





然后，打开命令行输入mvn，我们会看到mvn命令的运行效果

1. ~ C:\Users\Administrator>mvn
2. [INFO] Scanning for projects...
3. [INFO] ------------------------------------------------------------------------
4. [INFO] BUILD FAILURE
5. [INFO] ------------------------------------------------------------------------
6. [INFO] Total time: 0.086s
7. [INFO] Finished at: Mon Sep 30 18:26:58 CST 2013
8. [INFO] Final Memory: 2M/179M
9. [INFO] ------------------------------------------------------------------------
10. [ERROR] No goals have been specified for this build. You must specify a valid lifecycle phase or a goal in the format : or :[:]:. Available lifecycle phases are: validate, initialize, generate-sources, process-sources, generate-resources, process-resources, compile, process-class
11. es, generate-test-sources, process-test-sources, generate-test-resources, process-test-resources, test-compile, process-test-classes, test, prepare-package, package, pre-integration-test, integration-test, post-integration-test, verify, install, deploy, pre-clean, clean, post-clean, pre-site, site, post-site, site-deploy. -> [Help 1]
12. [ERROR]
13. [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
14. [ERROR] Re-run Maven using the -X switch to enable full debug logging.
15. [ERROR]
16. [ERROR] For more information about the errors and possible solutions, please read the following articles:
17. [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/NoGoalSpecifiedException

复制代码

安装Eclipse的Maven插件：[Maven Integration for Eclipse](http://www.eclipse.org/m2e/)

Maven的Eclipse插件配置

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/203954j6qt7txmie7q1zmo.png)](http://blog.fens.me/wp-content/uploads/2013/09/eclipse-maven.png)





\3. Hadoop开发环境介绍

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/203954m5xvj5qd5xj0w5ht.png)](http://blog.fens.me/wp-content/uploads/2013/09/hadoop-dev.png)





如上图所示，我们可以选择在win中开发，也可以在linux中开发，本地启动Hadoop或者远程调用Hadoop，标配的工具都是Maven和Eclipse。

Hadoop集群系统环境：

- Linux: Ubuntu 12.04.2 LTS 64bit Server
- Java: 1.6.0_29
- Hadoop: hadoop-1.0.3，单节点，IP:192.168.1.210

\4. 用Maven构建Hadoop环境

- \1. 用Maven创建一个标准化的Java项目
- \2. 导入项目到eclipse
- \3. 增加hadoop依赖，修改pom.xml
- \4. 下载依赖
- \5. 从Hadoop集群环境下载hadoop配置文件
- \6. 配置本地host

**1). 用Maven创建一个标准化的Java项目**

1. 
2. ~ D:\workspace\java>mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DgroupId=org.conan.myhadoop.mr
3. -DartifactId=myHadoop -DpackageName=org.conan.myhadoop.mr -Dversion=1.0-SNAPSHOT -DinteractiveMode=false
4. [INFO] Scanning for projects...
5. [INFO]
6. [INFO] ------------------------------------------------------------------------
7. [INFO] Building Maven Stub Project (No POM) 1
8. [INFO] ------------------------------------------------------------------------
9. [INFO]
10. [INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
11. [INFO]
12. [INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
13. [INFO]
14. [INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
15. [INFO] Generating project in Batch mode
16. [INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.
17. 0)
18. Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/1.0/maven-archet
19. ype-quickstart-1.0.jar
20. Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/1.0/maven-archety
21. pe-quickstart-1.0.jar (5 KB at 4.3 KB/sec)
22. Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/1.0/maven-archet
23. ype-quickstart-1.0.pom
24. Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/1.0/maven-archety
25. pe-quickstart-1.0.pom (703 B at 1.6 KB/sec)
26. [INFO] ----------------------------------------------------------------------------
27. [INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-quickstart:1.0
28. [INFO] ----------------------------------------------------------------------------
29. [INFO] Parameter: groupId, Value: org.conan.myhadoop.mr
30. [INFO] Parameter: packageName, Value: org.conan.myhadoop.mr
31. [INFO] Parameter: package, Value: org.conan.myhadoop.mr
32. [INFO] Parameter: artifactId, Value: myHadoop
33. [INFO] Parameter: basedir, Value: D:\workspace\java
34. [INFO] Parameter: version, Value: 1.0-SNAPSHOT
35. [INFO] project created from Old (1.x) Archetype in dir: D:\workspace\java\myHadoop
36. [INFO] ------------------------------------------------------------------------
37. [INFO] BUILD SUCCESS
38. [INFO] ------------------------------------------------------------------------
39. [INFO] Total time: 8.896s
40. [INFO] Finished at: Sun Sep 29 20:57:07 CST 2013
41. [INFO] Final Memory: 9M/179M
42. [INFO] ------------------------------------------------------------------------

复制代码

进入项目，执行mvn命令



1. ~ D:\workspace\java>cd myHadoop
2. ~ D:\workspace\java\myHadoop>mvn clean install
3. [INFO]
4. [INFO] --- maven-jar-plugin:2.3.2:jar (default-jar) @ myHadoop ---
5. [INFO] Building jar: D:\workspace\java\myHadoop\target\myHadoop-1.0-SNAPSHOT.jar
6. [INFO]
7. [INFO] --- maven-install-plugin:2.3.1:install (default-install) @ myHadoop ---
8. [INFO] Installing D:\workspace\java\myHadoop\target\myHadoop-1.0-SNAPSHOT.jar to C:\Users\Administrator\.m2\repository\o
9. rg\conan\myhadoop\mr\myHadoop\1.0-SNAPSHOT\myHadoop-1.0-SNAPSHOT.jar
10. [INFO] Installing D:\workspace\java\myHadoop\pom.xml to C:\Users\Administrator\.m2\repository\org\conan\myhadoop\mr\myHa
11. doop\1.0-SNAPSHOT\myHadoop-1.0-SNAPSHOT.pom
12. [INFO] ------------------------------------------------------------------------
13. [INFO] BUILD SUCCESS
14. [INFO] ------------------------------------------------------------------------
15. [INFO] Total time: 4.348s
16. [INFO] Finished at: Sun Sep 29 20:58:43 CST 2013
17. [INFO] Final Memory: 11M/179M
18. [INFO] ------------------------------------------------------------------------

复制代码

**2). 导入项目到eclipse**

我们创建好了一个基本的maven项目，然后导入到eclipse中。 这里我们最好已安装好了Maven的插件。

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/204137dwss6vy9wsfyvqqw.png)](http://blog.fens.me/wp-content/uploads/2013/09/hadoop-eclipse.png)





**3). 增加hadoop依赖**

这里我使用hadoop-1.0.3版本，修改文件：pom.xml

1. ~ vi pom.xml
2. 
3. <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
4. xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
5. <modelVersion>4.0.0</modelVersion>
6. <groupId>org.conan.myhadoop.mr</groupId>
7. <artifactId>myHadoop</artifactId>
8. <packaging>jar</packaging>
9. <version>1.0-SNAPSHOT</version>
10. <name>myHadoop</name>
11. <url>http://maven.apache.org</url>
12. <dependencies>
13. <dependency>
14. <groupId>org.apache.hadoop</groupId>
15. <artifactId>hadoop-core</artifactId>
16. <version>1.0.3</version>
17. </dependency>
18. 
19. <dependency>
20. <groupId>junit</groupId>
21. <artifactId>junit</artifactId>
22. <version>4.4</version>
23. <scope>test</scope>
24. </dependency>
25. </dependencies>
26. </project>

复制代码

**4). 下载依赖**

下载依赖：

1. ~ mvn clean install

复制代码

在eclipse中刷新项目：

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/204328w7m50ygzfmaoogoo.png)](http://blog.fens.me/wp-content/uploads/2013/09/hadoop-eclipse-maven.png)





项目的依赖程序，被自动加载的库路径下面。

**5). 从Hadoop集群环境下载hadoop配置文件**

- - core-site.xml
  - hdfs-site.xml
  - mapred-site.xml

查看core-site.xml

1. <?xml version="1.0"?>
2. <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
3. 
4. <configuration>
5. <property>
6. <name>fs.default.name</name>
7. <value>hdfs://master:9000</value>
8. </property>
9. <property>
10. <name>hadoop.tmp.dir</name>
11. <value>/home/conan/hadoop/tmp</value>
12. </property>
13. <property>
14. <name>io.sort.mb</name>
15. <value>256</value>
16. </property>
17. </configuration>

复制代码

查看hdfs-site.xml

1. <?xml version="1.0"?>
2. <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
3. 
4. <configuration>
5. <property>
6. <name>dfs.data.dir</name>
7. <value>/home/conan/hadoop/data</value>
8. </property>
9. <property>
10. <name>dfs.replication</name>
11. <value>1</value>
12. </property>
13. <property>
14. <name>dfs.permissions</name>
15. <value>false</value>
16. </property>
17. </configuration>

复制代码

查看mapred-site.xml



1. <?xml version="1.0"?>
2. <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
3. 
4. <configuration>
5. <property>
6. <name>mapred.job.tracker</name>
7. <value>hdfs://master:9001</value>
8. </property>
9. </configuration>

复制代码

保存在src/main/resources/hadoop目录下面

[![img](http://www.aboutyun.com/data/attachment/forum/201402/06/204509go6odocuzcoe6bjc.png)](http://blog.fens.me/wp-content/uploads/2013/09/hadoop-config.png)





删除原自动生成的文件：App.java和AppTest.java

**6).配置本地host，增加master的域名指向**

1. 
2. ~ vi c:/Windows/System32/drivers/etc/hosts
3. 
4. 192.168.1.210 master

复制代码

\6. MapReduce程序开发

编写一个简单的MapReduce程序，实现wordcount功能。

新一个Java文件：WordCount.java

1. package org.conan.myhadoop.mr;
2. 
3. import java.io.IOException;
4. import java.util.Iterator;
5. import java.util.StringTokenizer;
6. 
7. import org.apache.hadoop.fs.Path;
8. import org.apache.hadoop.io.IntWritable;
9. import org.apache.hadoop.io.Text;
10. import org.apache.hadoop.mapred.FileInputFormat;
11. import org.apache.hadoop.mapred.FileOutputFormat;
12. import org.apache.hadoop.mapred.JobClient;
13. import org.apache.hadoop.mapred.JobConf;
14. import org.apache.hadoop.mapred.MapReduceBase;
15. import org.apache.hadoop.mapred.Mapper;
16. import org.apache.hadoop.mapred.OutputCollector;
17. import org.apache.hadoop.mapred.Reducer;
18. import org.apache.hadoop.mapred.Reporter;
19. import org.apache.hadoop.mapred.TextInputFormat;
20. import org.apache.hadoop.mapred.TextOutputFormat;
21. 
22. public class WordCount {
23. 
24. ​    public static class WordCountMapper extends MapReduceBase implements Mapper<Object, Text, Text, IntWritable> {
25. ​        private final static IntWritable one = new IntWritable(1);
26. ​        private Text word = new Text();
27. 
28. ​        @Override
29. ​        public void map(Object key, Text value, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
30. ​            StringTokenizer itr = new StringTokenizer(value.toString());
31. ​            while (itr.hasMoreTokens()) {
32. ​                word.set(itr.nextToken());
33. ​                output.collect(word, one);
34. ​            }
35. 
36. ​        }
37. ​    }
38. 
39. ​    public static class WordCountReducer extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable> {
40. ​        private IntWritable result = new IntWritable();
41. 
42. ​        @Override
43. ​        public void reduce(Text key, Iterator values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
44. ​            int sum = 0;
45. ​            while (values.hasNext()) {
46. ​                sum += values.next().get();
47. ​            }
48. ​            result.set(sum);
49. ​            output.collect(key, result);
50. ​        }
51. ​    }
52. 
53. ​    public static void main(String[] args) throws Exception {
54. ​        String input = "hdfs://192.168.1.210:9000/user/hdfs/o_t_account";
55. ​        String output = "hdfs://192.168.1.210:9000/user/hdfs/o_t_account/result";
56. 
57. ​        JobConf conf = new JobConf(WordCount.class);
58. ​        conf.setJobName("WordCount");
59. ​        conf.addResource("classpath:/hadoop/core-site.xml");
60. ​        conf.addResource("classpath:/hadoop/hdfs-site.xml");
61. ​        conf.addResource("classpath:/hadoop/mapred-site.xml");
62. 
63. ​        conf.setOutputKeyClass(Text.class);
64. ​        conf.setOutputValueClass(IntWritable.class);
65. 
66. ​        conf.setMapperClass(WordCountMapper.class);
67. ​        conf.setCombinerClass(WordCountReducer.class);
68. ​        conf.setReducerClass(WordCountReducer.class);
69. 
70. ​        conf.setInputFormat(TextInputFormat.class);
71. ​        conf.setOutputFormat(TextOutputFormat.class);
72. 
73. ​        FileInputFormat.setInputPaths(conf, new Path(input));
74. ​        FileOutputFormat.setOutputPath(conf, new Path(output));
75. 
76. ​        JobClient.runJob(conf);
77. ​        System.exit(0);
78. ​    }
79. 
80. }

复制代码

**启动Java APP.**

控制台错误

1. 2013-9-30 19:25:02 org.apache.hadoop.util.NativeCodeLoader 
2. 警告: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
3. 2013-9-30 19:25:02 org.apache.hadoop.security.UserGroupInformation doAs
4. 严重: PriviledgedActionException as:Administrator cause:java.io.IOException: Failed to set permissions of path: \tmp\hadoop-Administrator\mapred\staging\Administrator1702422322\.staging to 0700
5. Exception in thread "main" java.io.IOException: Failed to set permissions of path: \tmp\hadoop-Administrator\mapred\staging\Administrator1702422322\.staging to 0700
6. ​        at org.apache.hadoop.fs.FileUtil.checkReturnValue(FileUtil.java:689)
7. ​        at org.apache.hadoop.fs.FileUtil.setPermission(FileUtil.java:662)
8. ​        at org.apache.hadoop.fs.RawLocalFileSystem.setPermission(RawLocalFileSystem.java:509)
9. ​        at org.apache.hadoop.fs.RawLocalFileSystem.mkdirs(RawLocalFileSystem.java:344)
10. ​        at org.apache.hadoop.fs.FilterFileSystem.mkdirs(FilterFileSystem.java:189)
11. ​        at org.apache.hadoop.mapreduce.JobSubmissionFiles.getStagingDir(JobSubmissionFiles.java:116)
12. ​        at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:856)
13. ​        at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:850)
14. ​        at java.security.AccessController.doPrivileged(Native Method)
15. ​        at javax.security.auth.Subject.doAs(Subject.java:396)
16. ​        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1121)
17. ​        at org.apache.hadoop.mapred.JobClient.submitJobInternal(JobClient.java:850)
18. ​        at org.apache.hadoop.mapred.JobClient.submitJob(JobClient.java:824)
19. ​        at org.apache.hadoop.mapred.JobClient.runJob(JobClient.java:1261)
20. ​        at org.conan.myhadoop.mr.WordCount.main(WordCount.java:78)</font></font></font>

复制代码

这个错误是win中开发特有的错误，文件权限问题，在Linux下可以正常运行。

解决方法是，修改/hadoop-1.0.3/src/core/org/apache/hadoop/fs/FileUtil.java文件

688-692行注释，然后重新编译源代码，重新打一个hadoop.jar的包。

1. 
2. 685 private static void checkReturnValue(boolean rv, File p,
3. 686                                        FsPermission permission
4. 687                                        ) throws IOException {
5. 688     /*if (!rv) {
6. 689       throw new IOException("Failed to set permissions of path: " + p +
7. 690                             " to " +
8. 691                             String.format("%04o", permission.toShort()));
9. 692     }*/
10. 693   }</font></font></font>

复制代码

我这里自己打了一个hadoop-core-1.0.3.jar包，放到了lib下面。

我们还要替换maven中的hadoop类库。

1. ~ cp lib/hadoop-core-1.0.3.jar C:\Users\Administrator\.m2\repository\org\apache\hadoop\hadoop-core\1.0.3\hadoop-core-1.0.3.jar
2. 
3. 

复制代码

再次启动Java APP，控制台输出：

1. 2013-9-30 19:50:49 org.apache.hadoop.util.NativeCodeLoader 
2. 警告: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
3. 2013-9-30 19:50:49 org.apache.hadoop.mapred.JobClient copyAndConfigureFiles
4. 警告: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
5. 2013-9-30 19:50:49 org.apache.hadoop.mapred.JobClient copyAndConfigureFiles
6. 警告: No job jar file set.  User classes may not be found. See JobConf(Class) or JobConf#setJar(String).
7. 2013-9-30 19:50:49 org.apache.hadoop.io.compress.snappy.LoadSnappy 
8. 警告: Snappy native library not loaded
9. 2013-9-30 19:50:49 org.apache.hadoop.mapred.FileInputFormat listStatus
10. 信息: Total input paths to process : 4
11. 2013-9-30 19:50:50 org.apache.hadoop.mapred.JobClient monitorAndPrintJob
12. 信息: Running job: job_local_0001
13. 2013-9-30 19:50:50 org.apache.hadoop.mapred.Task initialize
14. 信息:  Using ResourceCalculatorPlugin : null
15. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask runOldMapper
16. 信息: numReduceTasks: 1
17. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
18. 信息: io.sort.mb = 100
19. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
20. 信息: data buffer = 79691776/99614720
21. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
22. 信息: record buffer = 262144/327680
23. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask$MapOutputBuffer flush
24. 信息: Starting flush of map output
25. 2013-9-30 19:50:50 org.apache.hadoop.mapred.MapTask$MapOutputBuffer sortAndSpill
26. 信息: Finished spill 0
27. 2013-9-30 19:50:50 org.apache.hadoop.mapred.Task done
28. 信息: Task:attempt_local_0001_m_000000_0 is done. And is in the process of commiting
29. 2013-9-30 19:50:51 org.apache.hadoop.mapred.JobClient monitorAndPrintJob
30. 信息:  map 0% reduce 0%
31. 2013-9-30 19:50:53 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
32. 信息: hdfs://192.168.1.210:9000/user/hdfs/o_t_account/part-m-00003:0+119
33. 2013-9-30 19:50:53 org.apache.hadoop.mapred.Task sendDone
34. 信息: Task 'attempt_local_0001_m_000000_0' done.
35. 2013-9-30 19:50:53 org.apache.hadoop.mapred.Task initialize
36. 信息:  Using ResourceCalculatorPlugin : null
37. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask runOldMapper
38. 信息: numReduceTasks: 1
39. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
40. 信息: io.sort.mb = 100
41. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
42. 信息: data buffer = 79691776/99614720
43. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
44. 信息: record buffer = 262144/327680
45. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask$MapOutputBuffer flush
46. 信息: Starting flush of map output
47. 2013-9-30 19:50:53 org.apache.hadoop.mapred.MapTask$MapOutputBuffer sortAndSpill
48. 信息: Finished spill 0
49. 2013-9-30 19:50:53 org.apache.hadoop.mapred.Task done
50. 信息: Task:attempt_local_0001_m_000001_0 is done. And is in the process of commiting
51. 2013-9-30 19:50:54 org.apache.hadoop.mapred.JobClient monitorAndPrintJob
52. 信息:  map 100% reduce 0%
53. 2013-9-30 19:50:56 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
54. 信息: hdfs://192.168.1.210:9000/user/hdfs/o_t_account/part-m-00000:0+113
55. 2013-9-30 19:50:56 org.apache.hadoop.mapred.Task sendDone
56. 信息: Task 'attempt_local_0001_m_000001_0' done.
57. 2013-9-30 19:50:56 org.apache.hadoop.mapred.Task initialize
58. 信息:  Using ResourceCalculatorPlugin : null
59. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask runOldMapper
60. 信息: numReduceTasks: 1
61. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
62. 信息: io.sort.mb = 100
63. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
64. 信息: data buffer = 79691776/99614720
65. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
66. 信息: record buffer = 262144/327680
67. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask$MapOutputBuffer flush
68. 信息: Starting flush of map output
69. 2013-9-30 19:50:56 org.apache.hadoop.mapred.MapTask$MapOutputBuffer sortAndSpill
70. 信息: Finished spill 0
71. 2013-9-30 19:50:56 org.apache.hadoop.mapred.Task done
72. 信息: Task:attempt_local_0001_m_000002_0 is done. And is in the process of commiting
73. 2013-9-30 19:50:59 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
74. 信息: hdfs://192.168.1.210:9000/user/hdfs/o_t_account/part-m-00001:0+110
75. 2013-9-30 19:50:59 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
76. 信息: hdfs://192.168.1.210:9000/user/hdfs/o_t_account/part-m-00001:0+110
77. 2013-9-30 19:50:59 org.apache.hadoop.mapred.Task sendDone
78. 信息: Task 'attempt_local_0001_m_000002_0' done.
79. 2013-9-30 19:50:59 org.apache.hadoop.mapred.Task initialize
80. 信息:  Using ResourceCalculatorPlugin : null
81. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask runOldMapper
82. 信息: numReduceTasks: 1
83. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
84. 信息: io.sort.mb = 100
85. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
86. 信息: data buffer = 79691776/99614720
87. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask$MapOutputBuffer 
88. 信息: record buffer = 262144/327680
89. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask$MapOutputBuffer flush
90. 信息: Starting flush of map output
91. 2013-9-30 19:50:59 org.apache.hadoop.mapred.MapTask$MapOutputBuffer sortAndSpill
92. 信息: Finished spill 0
93. 2013-9-30 19:50:59 org.apache.hadoop.mapred.Task done
94. 信息: Task:attempt_local_0001_m_000003_0 is done. And is in the process of commiting
95. 2013-9-30 19:51:02 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
96. 信息: hdfs://192.168.1.210:9000/user/hdfs/o_t_account/part-m-00002:0+79
97. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Task sendDone
98. 信息: Task 'attempt_local_0001_m_000003_0' done.
99. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Task initialize
100. 信息:  Using ResourceCalculatorPlugin : null
101. 2013-9-30 19:51:02 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
102. 信息: 
103. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Merger$MergeQueue merge
104. 信息: Merging 4 sorted segments
105. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Merger$MergeQueue merge
106. 信息: Down to the last merge-pass, with 4 segments left of total size: 442 bytes
107. 2013-9-30 19:51:02 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
108. 信息: 
109. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Task done
110. 信息: Task:attempt_local_0001_r_000000_0 is done. And is in the process of commiting
111. 2013-9-30 19:51:02 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
112. 信息: 
113. 2013-9-30 19:51:02 org.apache.hadoop.mapred.Task commit
114. 信息: Task attempt_local_0001_r_000000_0 is allowed to commit now
115. 2013-9-30 19:51:02 org.apache.hadoop.mapred.FileOutputCommitter commitTask
116. 信息: Saved output of task 'attempt_local_0001_r_000000_0' to hdfs://192.168.1.210:9000/user/hdfs/o_t_account/result
117. 2013-9-30 19:51:05 org.apache.hadoop.mapred.LocalJobRunner$Job statusUpdate
118. 信息: reduce > reduce
119. 2013-9-30 19:51:05 org.apache.hadoop.mapred.Task sendDone
120. 信息: Task 'attempt_local_0001_r_000000_0' done.
121. 2013-9-30 19:51:06 org.apache.hadoop.mapred.JobClient monitorAndPrintJob
122. 信息:  map 100% reduce 100%
123. 2013-9-30 19:51:06 org.apache.hadoop.mapred.JobClient monitorAndPrintJob
124. 信息: Job complete: job_local_0001
125. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
126. 信息: Counters: 20
127. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
128. 信息:   File Input Format Counters 
129. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
130. 信息:     Bytes Read=421
131. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
132. 信息:   File Output Format Counters 
133. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
134. 信息:     Bytes Written=348
135. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
136. 信息:   FileSystemCounters
137. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
138. 信息:     FILE_BYTES_READ=7377
139. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
140. 信息:     HDFS_BYTES_READ=1535
141. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
142. 信息:     FILE_BYTES_WRITTEN=209510
143. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
144. 信息:     HDFS_BYTES_WRITTEN=348
145. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
146. 信息:   Map-Reduce Framework
147. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
148. 信息:     Map output materialized bytes=458
149. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
150. 信息:     Map input records=11
151. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
152. 信息:     Reduce shuffle bytes=0
153. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
154. 信息:     Spilled Records=30
155. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
156. 信息:     Map output bytes=509
157. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
158. 信息:     Total committed heap usage (bytes)=1838546944
159. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
160. 信息:     Map input bytes=421
161. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
162. 信息:     SPLIT_RAW_BYTES=452
163. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
164. 信息:     Combine input records=22
165. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
166. 信息:     Reduce input records=15
167. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
168. 信息:     Reduce input groups=13
169. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
170. 信息:     Combine output records=15
171. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
172. 信息:     Reduce output records=13
173. 2013-9-30 19:51:06 org.apache.hadoop.mapred.Counters log
174. 信息:     Map output records=22

复制代码

成功运行了wordcount程序，通过命令我们查看输出结果

1. ~ hadoop fs -ls hdfs://192.168.1.210:9000/user/hdfs/o_t_account/result
2. 
3. Found 2 items
4. -rw-r--r--   3 Administrator supergroup          0 2013-09-30 19:51 /user/hdfs/o_t_account/result/_SUCCESS
5. -rw-r--r--   3 Administrator supergroup        348 2013-09-30 19:51 /user/hdfs/o_t_account/result/part-00000
6. 
7. ~ hadoop fs -cat hdfs://192.168.1.210:9000/user/hdfs/o_t_account/result/part-00000
8. 
9. 1,abc@163.com,2013-04-22        1
10. 10,ade121@sohu.com,2013-04-23   1
11. 11,addde@sohu.com,2013-04-23    1
12. 17:21:24.0      5
13. 2,dedac@163.com,2013-04-22      1
14. 20:21:39.0      6
15. 3,qq8fed@163.com,2013-04-22     1
16. 4,qw1@163.com,2013-04-22        1
17. 5,af3d@163.com,2013-04-22       1
18. 6,ab34@163.com,2013-04-22       1
19. 7,q8d1@gmail.com,2013-04-23     1
20. 8,conan@gmail.com,2013-04-23    1
21. 9,adeg@sohu.com,2013-04-23      1

复制代码

这样，我们就实现了在win7中的开发，通过Maven构建Hadoop依赖环境，在Eclipse中开发MapReduce的程序，然后运行JavaAPP。Hadoop应用会自动把我们的MR程序打成jar包，再上传的远程的hadoop环境中运行，返回日志在Eclipse控制台输出。

\7. 模板项目上传github

<https://github.com/bsspirit/maven_hadoop_template>

大家可以下载这个项目，做为开发的起点。

1. ~ git clone https://github.com/bsspirit/maven_hadoop_template.git

复制代码

我们完成第一步，下面就将正式进入MapReduce开发实践。