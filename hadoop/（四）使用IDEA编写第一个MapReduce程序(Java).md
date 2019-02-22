# 使用IDEA编写第一个MapReduce程序(Java)

wordcount是Hadoop入门的经典例子，使用这个例子作为学习Hadoop的第一个程序。

本文使用Idea2018开发工具开发第一个Hadoop程序。使用的编程语言是Java。

打开idea，新建一个工程，如下图所示：

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103529211-1792217854.png)

在弹出新建工程的界面选择Java，接着选择SDK，一般默认即可，点击“Next”按钮，如下图：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103546311-1484609486.png)

在弹出的选择创建项目的模板页面，不做任何操作，直接点击“Next”按钮。

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103620692-2034743330.png)

输入项目名称，点击Finish，就完成了创建新项目的工作，我们的项目名称为：WordCount。如下图所示：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103630623-1435055799.png)

添加依赖jar包，和Eclipse一样，要给项目添加相关依赖包，否则会出错。

点击Idea的File菜单，然后点击“Project Structure”菜单，如下图所示：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103649945-76846475.png)

依次点击Modules和Dependencies，然后选择“+”的符号，如下图所示：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827103709764-1637042874.png)

 

选择hadoop的包，我用得是hadoop2.6.1。把下面的依赖包都加入到工程中，否则会出现某个类找不到的错误。

（1）”/usr/local/hadoop/share/hadoop/common”目录下的hadoop-common-2.6.1.jar和haoop-nfs-2.6.1.jar；

（2）/usr/local/hadoop/share/hadoop/common/lib”目录下的所有JAR包；

（3）“/usr/local/hadoop/share/hadoop/hdfs”目录下的haoop-hdfs-2.6.1.jar和haoop-hdfs-nfs-2.7.1.jar；

（4）“/usr/local/hadoop/share/hadoop/hdfs/lib”目录下的所有JAR包。

 

工程已经创建好，我们开始编写Map类、Reduce类和运行MapReduce的入口类：

 **JAVA编写MarReduce代码**

Map类如下：



```
 import org.apache.hadoop.io.IntWritable;

import org.apache.hadoop.io.LongWritable;

import org.apache.hadoop.io.Text;

import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;


public class WordcountMap extends Mapper<LongWritable,Text,Text,IntWritable> {
        public void map(LongWritable key,Text value,Context context)throws IOException,InterruptedException{

        String line = value.toString();//读取一行数据

        String str[] = line.split("");//因为英文字母是以“ ”为间隔的，因此使用“ ”分隔符将一行数据切成多个单词并存在数组中

        for(String s :str){//循环迭代字符串，将一个单词变成<key,value>形式，及<"hello",1>
            context.write(new Text(s),new IntWritable(1));
        }
    }
}
```



 

Reudce类：



```
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.io.Text;
import java.io.IOException;

public class WordcountReduce extends Reducer<Text,IntWritable,Text,IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values,Context context)throws IOException,InterruptedException{
        int count = 0;
        for(IntWritable value: values) {
            count++;
        }
        context.write(key,new IntWritable(count));
        }
}
```



 

 入口类 ：



```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;

public class WordCount {

    public static void main(String[] args)throws Exception{
        Configuration conf = new Configuration();
        //获取运行时输入的参数，一般是通过shell脚本文件传进来。
        String [] otherArgs = new         GenericOptionsParser(conf,args).getRemainingArgs();
        if(otherArgs.length < 2){
            System.err.println("必须输入读取文件路径和输出路径");
            System.exit(2);
        }
        Job job = new Job();
        job.setJarByClass(WordCount.class);
        job.setJobName("wordcount app");
    
        //设置读取文件的路径，都是从HDFS中读取。读取文件路径从脚本文件中传进来
        FileInputFormat.addInputPath(job,new Path(args[0]));
        //设置mapreduce程序的输出路径，MapReduce的结果都是输入到文件中
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

         //设置实现了map函数的类
        job.setMapperClass(WordcountMap.class);
        //设置实现了reduce函数的类
        job.setReducerClass(WordcountReduce.class);

         //设置reduce函数的key值
        job.setOutputKeyClass(Text.class);
        //设置reduce函数的value值
        job.setOutputValueClass(IntWritable.class);
        System.exit(job.waitForCompletion(true) ? 0 :1);
    }
}
```



 

代码写好之后，开始jar包，按照下图打包。点击“File”，然后点击“Project Structure”，弹出如下的界面，

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827104908512-344742563.png)

依次点击"Artifacts" -> "+" -> "JAR" -> "From modules with dependencies"，然后弹出一个选择入口类的界面，选择刚刚写好的WordCount类，如下图：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827104925211-554331483.png)

按照上面设置好之后，就开始打jar包，如下图：

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827104941212-2059852353.png)

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827104956042-299777192.png)

点击上图的“Build”之后就会生成一个jar包。jar的位置看下图，依次点击File->Project Structure->Artifacts就会看到如下的界面：

![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827105037533-573544066.png)

将打好包的wordcount.jar文件上传到装有hadoop集群的机器中，然后创建shell文件，shell文件内容如下，/usr/local/src/hadoop-2.6.1是hadoop集群中hadoop的安装位置，

```
/usr/local/src/hadoop-2.6.1/bin/hadoop jar wordcount.jar \ #执行jar文件的命令以及jar文件名，

hdfs://hadoop-master:8020/data/english.txt \ #输入路径

hdfs://hadoop-master:8020/wordcount_output #输出路径
```

 

执行shell文件之后，会看到如下的信息，

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827105147076-1040232905.png)

上图中数字1表示输入分片split的数量，数字2表示map和reduce的进度，数字3表示mapreduce执行成功，数字4表示启动多少个map任务，数字5表示启动多少个reduce任务。

自行成功后在hadoop集群中的hdfs文件系统中会看到一个wordcount_output的文件夹。使用“hadoop fs -ls /”命令查看：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827105206306-124776970.png)

在wordcount_output文件夹中有两个文件，分别是_SUCCESS和part-r-00000，part-r-00000记录着mapreduce的执行结果，使用hadoop fs -cat /wordcount_output/part-r-00000查看part-r-00000的内容：

 ![img](https://images2018.cnblogs.com/blog/110616/201808/110616-20180827105219516-725222392.png)

可以每个英文单词出现的次数。

至此，借助idea 2018工具开发第一个使用java语言编写的mapreduce程序已经成功执行。下面介绍使用python语言编写的第一个mapreduce程序，相对于java，python编写mapreduce会简单很多，因为hadoop提供streaming，streaming是使用Unix标准流作为Hadoop和应用程序之间的接口，所以可以使用任何语言通过标准输入输出来写MapReduce程序。