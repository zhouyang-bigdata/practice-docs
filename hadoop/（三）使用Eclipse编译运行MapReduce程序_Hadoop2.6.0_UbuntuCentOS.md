# 使用Eclipse编译运行MapReduce程序_Hadoop2.6.0_Ubuntu/CentOS

 虽然我们可以使用命令行编译打包运行自己的MapReduce程序，但毕竟编写代码不方便。使用 Eclipse，我们可以直接对 HDFS 中的文件进行操作，可以直接运行代码，省去许多繁琐的命令。



## 环境

本教程在 Hadoop 2.6.0 下验证通过，适用于 Ubuntu/CentOS 系统，理论上可用于任何原生 Hadoop 2 版本，如 Hadoop 2.4.1，Hadoop 2.7.1。

本教程主要测试环境：

- Ubuntu 14.04
- Hadoop 2.6.0（伪分布式）
- Eclipse 3.8

此外，本教材在 CentOS 6.4 系统中也验证通过，对 Ubuntu 与 CentOS 的不同配置之处有作出了注明。

## 安装 Eclipse

在 Ubuntu 和 CentOS 中安装 Eclipse 的方式有所不同，但之后的配置和使用是一样的。

在 Ubuntu 中安装 Eclipse，可从 Ubuntu 的软件中心直接搜索安装，在桌面左侧任务栏，点击“Ubuntu软件中心”。

![Ubuntu软件中心](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-01.png)Ubuntu软件中心

在右上角搜索栏中搜索 eclipse，在搜索结果中单击 eclipse，并点击安装。

![安装Eclipse](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-02.png)安装Eclipse

等待安装完成即可，Eclipse 的默认安装目录为：/usr/lib/eclipse。

在 CentOS 中安装 Eclipse，需要下载安装程序，我们选择 Eclipse IDE for Java Developers 版：

- 32位: <http://eclipse.bluemix.net/packages/mars.1/?JAVA-LINUX32>
- 64位: <http://eclipse.bluemix.net/packages/mars.1/?JAVA-LINUX64>

下载后执行如下命令，将 Eclipse 安装至 /usr/lib 目录中：

```bash
sudo tar -zxf ~/下载/eclipse-java-mars-1-linux-gtk*.tar.gz -C /usr/lib
```

Shell 命令

解压后即可使用。在 CentOS 中可以为程序创建桌面快捷方式，如下图所示，点击桌面右键，选择创建启动器，填写名称和程序位置（/usr/lib/eclipse/eclipse）：

![安装Eclipse](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-02-centos-link.png)安装Eclipse

## 安装 Hadoop-Eclipse-Plugin

要在 Eclipse 上编译和运行 MapReduce 程序，需要安装 hadoop-eclipse-plugin，可下载 Github 上的 [hadoop2x-eclipse-plugin](https://github.com/winghc/hadoop2x-eclipse-plugin)（备用下载地址：<http://pan.baidu.com/s/1i4ikIoP>）。

下载后，将 release 中的 hadoop-eclipse-kepler-plugin-2.6.0.jar （还提供了 2.2.0 和 2.4.1 版本）复制到 Eclipse 安装目录的 plugins 文件夹中，运行 `eclipse -clean` 重启 Eclipse 即可（添加插件后只需要运行一次该命令，以后按照正常方式启动就行了）。

```bash
unzip -qo ~/下载/hadoop2x-eclipse-plugin-master.zip -d ~/下载    # 解压到 ~/下载 中
sudo cp ~/下载/hadoop2x-eclipse-plugin-master/release/hadoop-eclipse-plugin-2.6.0.jar /usr/lib/eclipse/plugins/    # 复制到 eclipse 安装目录的 plugins 目录下
/usr/lib/eclipse/eclipse -clean    # 添加插件后需要用这种方式使插件生效
```

Shell 命令

## 配置 Hadoop-Eclipse-Plugin

**在继续配置前请确保已经开启了 Hadoop。**

启动 Eclipse 后就可以在左侧的Project Explorer中看到 DFS Locations（若看到的是 welcome 界面，点击左上角的 x 关闭就可以看到了。CentOS 需要切换 Perspective 后才能看到，即接下来配置步骤的第二步）。

![安装好Hadoop-Eclipse-Plugin插件后的效果](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-03.png)安装好Hadoop-Eclipse-Plugin插件后的效果

插件需要进一步的配置。

第一步：选择 Window 菜单下的 Preference。

![打开Preference](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-04.png)打开Preference

此时会弹出一个窗体，窗体的左侧会多出 Hadoop Map/Reduce 选项，点击此选项，选择 Hadoop 的安装目录（如/usr/local/hadoop，Ubuntu不好选择目录，直接输入就行）。

![选择 Hadoop 的安装目录](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-05.png)选择 Hadoop 的安装目录

第二步：切换 Map/Reduce 开发视图，选择 Window 菜单下选择 Open Perspective -> Other（CentOS 是 Window -> Perspective -> Open Perspective -> Other），弹出一个窗体，从中选择 Map/Reduce 选项即可进行切换。

![切换 Map/Reduce 开发视图](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-06.png)切换 Map/Reduce 开发视图

第三步：建立与 Hadoop 集群的连接，点击 Eclipse软件右下角的 Map/Reduce Locations 面板，在面板中单击右键，选择 New Hadoop Location。

![建立与 Hadoop 集群的连接](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-07.png)建立与 Hadoop 集群的连接

在弹出来的 General 选项面板中，General 的设置要与 Hadoop 的配置一致。一般两个 Host 值是一样的，如果是伪分布式，填写 localhost 即可，另外我使用的Hadoop伪分布式配置，设置 fs.defaultFS 为 hdfs://localhost:9000，则 DFS Master 的 Port 要改为 9000。Map/Reduce(V2) Master 的 Port 用默认的即可，Location Name 随意填写。

最后的设置如下图所示：

![Hadoop Location 的设置](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-08.png)Hadoop Location 的设置

Advanced parameters 选项面板是对 Hadoop 参数进行配置，实际上就是填写 Hadoop 的配置项(/usr/local/hadoop/etc/hadoop中的配置文件)，如我配置了 hadoop.tmp.dir ，就要进行相应的修改。但修改起来会比较繁琐，我们可以通过复制配置文件的方式解决（下面会说到）。

总之，我们只要配置 General 就行了，点击 finish，Map/Reduce Location 就创建好了。

## 在 Eclipse 中操作 HDFS 中的文件

配置好后，点击左侧 Project Explorer 中的 MapReduce Location （点击三角形展开）就能直接查看 HDFS 中的文件列表了（HDFS 中要有文件，如下图是 WordCount 的输出结果），双击可以查看内容，右键点击可以上传、下载、删除 HDFS 中的文件，无需再通过繁琐的 hdfs dfs -ls 等命令进行操作了。
以下output/part-r-00000文件记录了输出结果。

![使用Eclipse查看HDFS中的文件内容](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-10.png)使用Eclipse查看HDFS中的文件内容

如果无法查看，可右键点击 Location 尝试 Reconnect 或重启 Eclipse。

Tips

HDFS 中的内容变动后，Eclipse 不会同步刷新，需要右键点击 Project Explorer中的 MapReduce Location，选择 Refresh，才能看到变动后的文件。

## 在 Eclipse 中创建 MapReduce 项目

点击 File 菜单，选择 New -> Project…:

![创建Project](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-11.png)创建Project

选择 Map/Reduce Project，点击 Next。

![创建MapReduce项目](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-12.png)创建MapReduce项目

填写 Project name 为 WordCount 即可，点击 Finish 就创建好了项目。

![填写项目名](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-13.png)填写项目名

此时在左侧的 Project Explorer 就能看到刚才建立的项目了。

![项目创建完成](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-14.png)项目创建完成

接着右键点击刚创建的 WordCount 项目，选择 New -> Class

![新建Class](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-15.png)新建Class

需要填写两个地方：在 Package 处填写 org.apache.hadoop.examples；在 Name 处填写 WordCount。

![填写Class信息](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-16.png)填写Class信息

创建 Class 完成后，在 Project 的 src 中就能看到 WordCount.java 这个文件。将如下 WordCount 的代码复制到该文件中。

```java
package org.apache.hadoop.examples;
 
import java.io.IOException;
import java.util.Iterator;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
 
public class WordCount {
    public WordCount() {
    }
 
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = (new GenericOptionsParser(conf, args)).getRemainingArgs();
        if(otherArgs.length < 2) {
            System.err.println("Usage: wordcount <in> [<in>...] <out>");
            System.exit(2);
        }
 
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(WordCount.TokenizerMapper.class);
        job.setCombinerClass(WordCount.IntSumReducer.class);
        job.setReducerClass(WordCount.IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
 
        for(int i = 0; i < otherArgs.length - 1; ++i) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
 
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[otherArgs.length - 1]));
        System.exit(job.waitForCompletion(true)?0:1);
    }
 
    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();
 
        public IntSumReducer() {
        }
 
        public void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
            int sum = 0;
 
            IntWritable val;
            for(Iterator i$ = values.iterator(); i$.hasNext(); sum += val.get()) {
                val = (IntWritable)i$.next();
            }
 
            this.result.set(sum);
            context.write(key, this.result);
        }
    }
 
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private static final IntWritable one = new IntWritable(1);
        private Text word = new Text();
 
        public TokenizerMapper() {
        }
 
        public void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
 
            while(itr.hasMoreTokens()) {
                this.word.set(itr.nextToken());
                context.write(this.word, one);
            }
 
        }
    }
}
```



## 通过 Eclipse 运行 MapReduce

在运行 MapReduce 程序前，还需要执行一项重要操作（也就是上面提到的通过复制配置文件解决参数设置问题）：将 /usr/local/hadoop/etc/hadoop 中将**有修改过的配置文件**（如伪分布式需要 core-site.xml 和 hdfs-site.xml），以及 log4j.properties 复制到 WordCount 项目下的 src 文件夹（~/workspace/WordCount/src）中：

```bash
cp /usr/local/hadoop/etc/hadoop/core-site.xml ~/workspace/WordCount/srccp /usr/local/hadoop/etc/hadoop/hdfs-site.xml ~/workspace/WordCount/srccp /usr/local/hadoop/etc/hadoop/log4j.properties ~/workspace/WordCount/src
```

Shell 命令

没有复制这些文件的话程序将无法正确运行，本教程最后再解释为什么需要复制这些文件。

复制完成后，务必右键点击 WordCount 选择 refresh 进行刷新（不会自动刷新，需要手动刷新），可以看到文件结构如下所示：

![WordCount项目文件结构](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-17.png)WordCount项目文件结构

点击工具栏中的 Run 图标，或者右键点击 Project Explorer 中的 WordCount.java，选择 Run As -> Run on Hadoop，就可以运行 MapReduce 程序了。不过由于没有指定参数，运行时会提示 “Usage: wordcount “，需要通过Eclipse设定一下运行参数。

右键点击刚创建的 WordCount.java，选择 Run As -> Run Configurations，在此处可以设置运行时的相关参数（如果 Java Application 下面没有 WordCount，那么需要先双击 Java Application）。切换到 “Arguments” 栏，在 Program arguments 处填写 “input output” 就可以了。

![设置程序运行参数](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-18.png)设置程序运行参数

或者也可以直接在代码中设置好输入参数。可将代码 main() 函数的 `String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();` 改为：

```java
// String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();String[] otherArgs=new String[]{"input","output"}; /* 直接设置输入参数 */
```

Java

设定参数后，再次运行程序，可以看到运行成功的提示，刷新 DFS Location 后也能看到输出的 output 文件夹。

![WordCount 运行结果](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2014/10/hadoop-build-project-using-eclipse-19.png)WordCount 运行结果

至此，你就可以使用 Eclipse 方便的进行 MapReduce程序的开发了。

## 在 Eclipse 中运行 MapReduce 程序会遇到的问题

在使用 Eclipse 运行 MapReduce 程序时，会读取 Hadoop-Eclipse-Plugin 的 Advanced parameters 作为 Hadoop 运行参数，如果我们未进行修改，则默认的参数其实就是单机（非分布式）参数，因此程序运行时是读取本地目录而不是 HDFS 目录，就会提示 Input 路径不存在。

```
Exception in thread "main" org.apache.hadoop.mapreduce.lib.input.InvalidInputException: Input path does not exist: file:/home/hadoop/workspace/WordCountProject/input
```

所以我们需要将配置文件复制到项目中的 src 目录，来覆盖这些参数。让程序能够正确运行。

log4j 用于记录程序的输出日记，需要 log4j.properties 这个配置文件，如果没有复制该文件到项目中，运行程序后在 Console 面板中会出现警告提示：

```
log4j:WARN No appenders could be found for logger (org.apache.hadoop.metrics2.lib.MutableMetricsFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

虽然不影响程序的正确运行的，但程序运行时无法看到任何提示消息（只能看到出错信息）。

## 参考资料

- <http://www.cnblogs.com/xia520pi/archive/2012/05/20/2510723.html>
- <http://www.blogjava.net/LittleRain/archive/2006/12/31/91165.html>

