# Hadoop数据压缩

### **1****.1 概述**

压缩技术能够有效减少底层存储系统（HDFS）读写字节数。压缩提高了网络带宽和磁盘空间的效率。在Hadood下，尤其是数据规模很大和工作负载密集的情况下，使用数据压缩显得非常重要。在这种情况下，I/O操作和网络数据传输要花大量的时间。还有，Shuffle与Merge过程同样也面临着巨大的I/O压力。

鉴于磁盘I/O和网络带宽是Hadoop的宝贵资源，数据压缩对于节省资源、最小化磁盘I/O和网络传输非常有帮助。不过，尽管压缩与解压操作的CPU开销不高，其性能的提升和资源的节省并非没有代价。

如果磁盘I/O和网络带宽影响了MapReduce作业性能，在任意MapReduce阶段启用压缩都可以改善端到端处理时间并减少I/O和网络流量。

压缩**mapreduce的一种优化策略：通过压缩编码对mapper或者reducer的输出进行压缩，以减少磁盘IO，**提高MR程序运行速度（但相应增加了cpu运算负担）。

注意：压缩特性运用得当能提高性能，但运用不当也可能降低性能。

基本原则：

（1）运算密集型的job，少用压缩

（2）IO密集型的job，多用压缩

### **1****.2 MR支持的压缩编码**

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFAULT  | 是，直接使用 | DEFAULT | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFAULT | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 否，需要安装 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

 压缩性能的比较

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

<http://google.github.io/snappy/>

On a single core of a Core i7 processor in 64-bit mode, Snappy compresses at about 250 MB/sec or more and decompresses at about 500 MB/sec or more.

### **1.****3** **各种压缩****方式详解**

### **1.3****.****1** **Gzip压缩**

优点：压缩率比较高，而且压缩/解压速度也比较快；hadoop本身支持，在应用中处理gzip格式的文件就和直接处理文本一样；大部分linux系统都自带gzip命令，使用方便。

缺点：不支持split。

应用场景：当每个文件压缩之后在130M以内的（1个块大小内），都可以考虑用gzip压缩格式。譬如说一天或者一个小时的日志压缩成一个gzip文件，运行mapreduce程序的时候通过多个gzip文件达到并发。hive程序，streaming程序，和java写的mapreduce程序完全和文本处理一样，压缩之后原来的程序不需要做任何修改。

### **1.3.2** **lzo压缩**

优点：压缩/解压速度也比较快，合理的压缩率；支持split，是hadoop中最流行的压缩格式；可以在linux系统下安装lzop命令，使用方便。

缺点：压缩率比gzip要低一些；hadoop本身不支持，需要安装；在应用中对lzo格式的文件需要做一些特殊处理（为了支持split需要建索引，还需要指定inputformat为lzo格式）。

应用场景：一个很大的文本文件，压缩之后还大于200M以上的可以考虑，而且单个文件越大，lzo优点越越明显。

### **1.3.3** **snappy压缩**

优点：高速压缩速度和合理的压缩率。

缺点：不支持split；压缩率比gzip要低；hadoop本身不支持，需要安装； 

应用场景：当mapreduce作业的map输出的数据比较大的时候，作为map到reduce的中间数据的压缩格式；或者作为一个mapreduce作业的输出和另外一个mapreduce作业的输入。

### **1.3.4** **bzip2压缩**

优点：支持split；具有很高的压缩率，比gzip压缩率都高；hadoop本身支持，但不支持native；在linux系统下自带bzip2命令，使用方便。

缺点：压缩/解压速度慢；不支持native。

应用场景：适合对速度要求不高，但需要较高的压缩率的时候，可以作为mapreduce作业的输出格式；或者输出之后的数据比较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用得比较少的情况；或者对单个很大的文本文件想压缩减少存储空间，同时又需要支持split，而且兼容之前的应用程序（即应用程序不需要修改）的情况。

### **1.3.5 如何选择压缩格式？**

Hadoop应用处理的数据集非常大，因此需要借助于压缩。使用哪种压缩格式与待处理的文件的大小、格式和所使用的工具相关。下面我们给出了一些建议，大致是按照效率从高到低排序的。

1）使用容器文件格式，例如顺序文件、RCFile或者Avro 数据文件，所有这些文件格式同时支持压缩和切分。通常最好与一个快速压缩工具联合使用，例如LZO，LZ4或者 Snappy。

 2）使用支持切分的压缩格式，例如bzip2(尽管bzip2 非常慢)，或者使用通过索引实现切分的压缩格式，例如LZO。

3）在应用中将文件切分成块，并使用任意一种压缩格式为每个数据块建立压缩文件(不论它是否支持切分)。这种情况下，需要合理选择数据块的大小，以确保压缩后数据块的大小近似与HDFS块的大小。

4）存储未经压缩的文件。

对大文件来说，不要使用不支持切分整个文件的压缩格式，因为会失去数据的本地特性，进而造成MapReduce应用效率低下。

### **1****.****4** **采用压缩****的****位置**

压缩可以在MapReduce作用的任意阶段启用。

 ![img](./Hadoop（八）Hadoop数据压缩与企业级优化 - Frankdeng - 博客园_files/1385722-20180519232212510-629145230.png)

1）输入压缩：

在有大量数据并计划重复处理的情况下，应该考虑对输入进行压缩。然而，你无须显示指定使用的编解码方式。Hadoop自动检查文件扩展名，如果扩展名能够匹配，就会用恰当的编解码方式对文件进行压缩和解压。否则，Hadoop就不会使用任何编解码器。

2）压缩mapper输出：

当map任务输出的中间数据量很大时，应考虑在此阶段采用压缩技术。这能显著改善内部数据Shuffle过程，而Shuffle过程在Hadoop处理过程中是资源消耗最多的环节。如果发现数据量大造成网络传输缓慢，应该考虑使用压缩技术。可用于压缩mapper输出的快速编解码器包括LZO或者Snappy。

注：LZO是供Hadoop压缩数据用的通用压缩编解码器。其设计目标是达到与硬盘读取速度相当的压缩速度，因此速度是优先考虑的因素，而不是压缩率。与gzip编解码器相比，它的压缩速度是gzip的5倍，而解压速度是gzip的2倍。同一个文件用LZO压缩后比用gzip压缩后大50%，但比压缩前小25%~50%。这对改善性能非常有利，map阶段完成时间快4倍。

3）压缩reducer输出：

在此阶段启用压缩技术能够减少要存储的数据量，因此降低所需的磁盘空间。当mapreduce作业形成作业链条时，因为第二个作业的输入也已压缩，所以启用压缩同样有效。

### **1****.****5** **压缩配置****参数**

要在Hadoop中启用压缩，可以配置如下参数（mapred-site.xml文件中）：

| 参数                                              | 默认值                                                       | 阶段        | 建议                                         |
| ------------------------------------------------- | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| io.compression.codecs   （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
| mapreduce.map.output.compress                     | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
| mapreduce.map.output.compress.codec               | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress        | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec  | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type   | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK  |

### **1****.****6** **压缩实战**

压缩案例详见压缩/解压缩。 

## 二 Hadoop企业优化

### **2.1 M****ap****R****educe跑慢****的原因**

Mapreduce 程序效率的瓶颈在于两点：

1）计算机性能

CPU、内存、磁盘健康、网络

2）I/O 操作优化

（1）数据倾斜

（2）map和reduce数设置不合理

（3）reduce等待过久

（4）小文件过多

（5）大量的不可分块的超大文件

（6）spill次数过多

（7）merge次数过多等。

### **2.2 M****ap****R****educe优化方法**

MapReduce优化方法主要从以下六个方面考虑：

### **2.2.1 数据输入**

（1）合并小文件：在执行mr任务前将小文件进行合并，大量的小文件会产生大量的map任务，增大map任务装载次数，而任务的装载比较耗时，从而导致 mr 运行较慢。

（2）采用ConbinFileInputFormat来作为输入，解决输入端大量小文件场景。

### **2.2.2 Map阶段**

1）减少spill次数：通过调整io.sort.mb及sort.spill.percent参数值，增大触发spill的内存上限，减少spill次数，从而减少磁盘 IO。

2）减少merge次数：通过调整io.sort.factor参数，增大merge的文件数目，减少merge的次数，从而缩短mr处理时间。

3）在 map 之后先进行combine处理，减少 I/O。

### **2.2.3 Reduce阶段**

1）合理设置map和reduce数：两个都不能设置太少，也不能设置太多。太少，会导致task等待，延长处理时间；太多，会导致 map、reduce任务间竞争资源，造成处理超时等错误。

2）设置map、reduce共存：调整slowstart.completedmaps参数，使map运行到一定程度后，reduce也开始运行，减少reduce的等待时间。

3）规避使用reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。

4）合理设置reduc端的buffer，默认情况下，数据达到一个阈值的时候，buffer中的数据就会写入磁盘，然后reduce会从磁盘中获得所有的数据。也就是说，buffer和reduce是没有直接关联的，中间多个一个写磁盘->读磁盘的过程，既然有这个弊端，那么就可以通过参数来配置，使得buffer中的一部分数据可以直接输送到reduce，从而减少IO开销：mapred.job.reduce.input.buffer.percent，默认为0.0。当值大于0的时候，会保留指定比例的内存读buffer中的数据直接拿给reduce使用。这样一来，设置buffer需要内存，读取数据需要内存，reduce计算也要内存，所以要根据作业的运行情况进行调整。

### **2.2.4 IO传输**

1）采用数据压缩的方式，减少网络IO的的时间。安装Snappy和LZOP压缩编码器。

2）使用SequenceFile二进制文件

### **2.2.5 数据倾斜问题**

1）数据倾斜现象

数据频率倾斜——某一个区域的数据量要远远大于其他区域。

数据大小倾斜——部分记录的大小远远大于平均值。

2）如何收集倾斜数据

在reduce方法中加入记录map输出键的详细情况的功能。



3）减少数据倾斜的方法

**方法1：抽样和范围分区**

可以通过对原始数据进行抽样得到的结果集来预设分区边界值。

**方法2：自定义分区**

另一个抽样和范围分区的替代方案是基于输出键的背景知识进行自定义分区。例如，如果map输出键的单词来源于一本书。其中大部分必然是省略词（stopword）。那么就可以将自定义分区将这部分省略词发送给固定的一部分reduce实例。而将其他的都发送给剩余的reduce实例。

**方法3：Combine**

使用Combine可以大量地减小数据频率倾斜和数据大小倾斜。在可能的情况下，combine的目的就是聚合并精简数据。

方法4：采用Map Join，尽量避免Reduce Join。

### **2.2.6 常用的调优参数**

1）资源相关参数

（1）以下参数是在用户自己的mr应用程序中配置就可以生效（mapred-default.xml）

| 配置参数                                      | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个Map Task可使用的资源上限（单位:MB），默认为1024。如果Map Task实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb                    | 一个Reduce Task可使用的资源上限（单位:MB），默认为1024。如果Reduce Task实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个Map task可使用的最多cpu core数目，默认值: 1              |
| mapreduce.reduce.cpu.vcores                   | 每个Reduce task可使用的最多cpu core数目，默认值: 1           |
| mapreduce.reduce.shuffle.parallelcopies       | 每个reduce去map中拿数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.merge.percent        | buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.shuffle.input.buffer.percent | buffer大小占reduce可用内存的比例。默认值0.7                  |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放buffer中的数据，默认值是0.0        |

 （2）应该在yarn启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

| 配置参数                                    | 参数说明                          |
| ------------------------------------------- | --------------------------------- |
| yarn.scheduler.minimum-allocation-mb   1024 | 给应用程序container分配的最小内存 |
| yarn.scheduler.maximum-allocation-mb   8192 | 给应用程序container分配的最大内存 |
| yarn.scheduler.minimum-allocation-vcores 1  | 每个container申请的最小CPU核数    |
| yarn.scheduler.maximum-allocation-vcores 32 | 每个container申请的最大CPU核数    |
| yarn.nodemanager.resource.memory-mb   8192  | 给containers分配的最大物理内存    |

 （3）shuffle性能优化的关键参数，应在yarn启动之前就配置好（mapred-default.xml）

| 配置参数                               | 参数说明                          |
| -------------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb   100        | shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent   0.8 | 环形缓冲区溢出的阈值，默认80%     |

 2）容错相关参数(mapreduce性能优化)

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该task处于block状态，可能是卡住了，也许永远会卡主，为了防止因为用户程序永远block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |

### **2.3 HDFS****小****文件****优化方法**

### **2.3.1 HDFS小文件弊端**

HDFS上每个文件都要在namenode上建立一个索引，这个索引的大小约为150byte，这样当小文件比较多的时候，就会产生很多的索引文件，一方面会大量占用namenode的内存空间，另一方面就是索引文件过大是的索引速度变慢。

### **2.3.2 解决方案**

1）Hadoop Archive:

 是一个高效地将小文件放入HDFS块中的文件存档工具，它能够将多个小文件打包成一个HAR文件，这样在减少namenode内存使用的同时。

2）Sequence file：

 sequence file由一系列的二进制key/value组成，如果key为文件名，value为文件内容，则可以将大批小文件合并成一个大文件。

3）CombineFileInputFormat：

  CombineFileInputFormat是一种新的inputformat，用于将多个文件合并成一个单独的split，另外，它会考虑数据的存储位置。

4）开启JVM重用

对于大量小文件Job，可以开启JVM重用会减少45%运行时间。

JVM重用理解：一个map运行一个jvm，重用的话，在一个map在jvm上运行完毕后，jvm继续运行其他jvm

具体设置：mapreduce.job.jvm.numtasks值在10-20之间。

## 三 MapReduce实战

3.1  [大数据技术之WordCount案例](https://www.cnblogs.com/frankdeng/p/9256254.html)

3.2  [大数据技术之流量汇总案例](https://www.cnblogs.com/frankdeng/p/9256252.html)

3.3  [大数据技术之辅助排序和二次排序案例（GroupingComparator）](https://www.cnblogs.com/frankdeng/p/9256249.html)

3.4  [大数据技术之MapReduce中多表合并案例](https://www.cnblogs.com/frankdeng/p/9256248.html)

3.5  [大数据技术之小文件处理（自定义InputFormat）](https://www.cnblogs.com/frankdeng/p/9256245.html)

3.6  [过滤日志及自定义日志输出路径（自定义OutputFormat）](https://www.cnblogs.com/frankdeng/p/9256215.html)

3.7  [大数据技术之日志清洗案例](https://www.cnblogs.com/frankdeng/p/9255766.html#top)

3.8  [大数据技术之倒排索引（多job串联)](https://www.cnblogs.com/frankdeng/p/9255927.html)

3.9  [大数据技术之找博客共同好友案例](https://www.cnblogs.com/frankdeng/p/9255931.html)

3.10  [大数据技术之压缩解压缩案例](https://www.cnblogs.com/frankdeng/p/9255935.html)

## 四 常见错误

1）导包容易出错。尤其Text.

2）Mapper中第一个输入的参数必须是LongWritable或者NullWritable，不可以是IntWritable.  报的错误是类型转换异常。

3）java.lang.Exception: java.io.IOException: Illegal partition for 13926435656 (4)，说明partition和reducetask个数没对上，调整reducetask个数。

4）如果分区数不是1，但是reducetask为1，是否执行分区过程。答案是：不执行分区过程。因为在maptask的源码中，执行分区的前提是先判断reduceNum个数是否大于1。不大于1肯定不执行。

5）在Windows环境编译的jar包导入到linux环境中运行，

hadoop jar wc.jar com.xyg.mapreduce.wordcount.WordCountDriver /user/root/ /user/root/output

报如下错误：

Exception in thread "main" java.lang.UnsupportedClassVersionError: com/root/mapreduce/wordcount/WordCountDriver : Unsupported major.minor version 52.0

原因是Windows环境用的jdk1.7，linux环境用的jdk1.8。

解决方案：统一jdk版本。

6）缓存pd.txt小文件案例中，报找不到pd.txt文件

原因：大部分为路径书写错误。还有就是要检查pd.txt.txt的问题。

7）报类型转换异常。

通常都是在驱动函数中设置map输出和最终输出时编写错误。

Map输出的key如果没有排序，也会报类型转换异常。