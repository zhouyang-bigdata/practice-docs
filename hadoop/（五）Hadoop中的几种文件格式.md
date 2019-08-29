# Hadoop中的几种文件格式



Hadoop中的文件格式大致上分为面向行和面向列两类：

- 面向行：同一行的数据存储在一起，即连续存储。SequenceFile,MapFile,Avro Datafile。采用这种方式，如果只需要访问行的一小部分数据，亦需要将整行读入内存，推迟序列化一定程度上可以缓解这个问题，但是从磁盘读取整行数据的开销却无法避免。面向行的存储适合于整行数据需要同时处理的情况。
- 面向列：整个文件被切割为若干列数据，每一列数据一起存储。Parquet , RCFile,ORCFile。面向列的格式使得读取数据时，可以跳过不需要的列，适合于只处于行的一小部分字段的情况。但是这种格式的读写需要更多的内存空间，因为需要缓存行在内存中（为了获取多行中的某一列）。同时不适合流式写入，因为一旦写入失败，当前文件无法恢复，而面向行的数据在写入失败时可以重新同步到最后一个同步点，所以Flume采用的是面向行的存储格式。

![logical-table.jpg-142.5kB](http://static.zybuluo.com/BrandonLin/eexzepzf0wuf29w2e7wllcja/logical-table.jpg)

![file.png-201kB](http://static.zybuluo.com/BrandonLin/b6q7z6ueqipvtrosz6gkzhi7/file.png)

下面介绍几种相关的文件格式，它们在Hadoop体系上被广泛使用：

# 1. SequenceFile

SequenceFile的文件结构如下：

![Sequence.png-272.5kB](http://static.zybuluo.com/BrandonLin/il5vgcjt388skd9el66qmoc1/Sequence.png)

根据是否压缩，以及采用记录压缩还是块压缩，存储格式有所不同：

- 不压缩： 
  按照记录长度、Key长度、Value程度、Key值、Value值依次存储。长度是指字节数。采用指定的Serialization进行序列化。
- Record压缩： 
  只有value被压缩，压缩的codec保存在Header中。
- Block压缩： 
  多条记录被压缩在一起，可以利用记录之间的相似性，更节省空间。Block前后都加入了同步标识。Block的最小值由`io.seqfile.compress.blocksize`属性设置。 
  ![block-compression.png-217.8kB](http://static.zybuluo.com/BrandonLin/ldhug9ebi8o223wj9nwjx9vz/block-compression.png)

# 2. MapFile

MapFile是SequenceFile的变种，在SequenceFile中加入索引并排序后就是MapFile。索引作为一个单独的文件存储，一般每个128个记录存储一个索引。索引可以被载入内存，用于快速查找。存放数据的文件根据Key定义的顺序排列。 
MapFile的记录必须按照顺序写入，否则抛出IOException。

MapFile的衍生类型：

- SetFile:特殊的MapFile，用于存储一序列Writable类型的Key。Key按照顺序写入。
- ArrayFile：Key为整数，代表在数组中的位置，value为Writable类型。
- BloomMapFile：针对MapFile的get()方法，使用动态Bloom过滤器进行优化。过滤器保存在内存中，只有带key值存在的时候，才会调用常规的get()方法，真正进行读操作。

Hadoop体系下面向列的文件包括RCFile，ORCFile，Parquet的。Avro的面向列版本为Trevni。

# 3. RCFile

Hive的Record Columnar File,这种类型的文件先将数据按行划分成Row Group，在Row Group内部，再将数据按列划分存储。其结构如下：

![rcfile.png-51.9kB](http://static.zybuluo.com/BrandonLin/ccqyzrejppyz3h6z0dn0lxsf/rcfile.png)

相比较于单纯地面向行和面向列：

![QQ截图20160801192417.png-55.6kB](http://static.zybuluo.com/BrandonLin/dix8emn8tsnu21dtjw1kjlxz/QQ%E6%88%AA%E5%9B%BE20160801192417.png)

![QQ截图20160801192358.png-47.5kB](http://static.zybuluo.com/BrandonLin/6uup6tigmgjjn4id61el2n9z/QQ%E6%88%AA%E5%9B%BE20160801192358.png)

更详细的介绍参考[RCFile论文](http://web.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-11-4.pdf)。

# 4. ORCFile

[RCFile](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC)（Optimized Record Columnar File)提供了一种比RCFile更加高效的文件格式。其内部将数据划分为默认大小为250M的Stripe。每个Stripe包括索引、数据和Footer。索引存储每一列的最大最小值，以及列中每一行的位置。

![OrcFileLayout.png-124.9kB](http://static.zybuluo.com/BrandonLin/ggbuozx12eauv8g0ypgehlln/OrcFileLayout.png)

在Hive中，如下命令用于使用ORCFile：

```
CREATE TABLE ... STORED AAS ORC
ALTER TABLE ... SET FILEFORMAT ORC
SET hive.default.fileformat=ORC123
```

# 5. Parquet

一种通用的面向列的存储格式，基于Google的Dremel。特别擅长处理深度嵌套的数据。

![parquet.png-237.6kB](http://static.zybuluo.com/BrandonLin/xvybcbth3d6ekje284f79g7h/parquet.png)

对于嵌套结构，Parquet将其转换为平面的列存储，嵌套结构通过Repeat Level和Definition Level来表示（R和D），在读取数据重构整条记录的时候，使用元数据重构记录的结构。下面是R和D的一个例子：

```json
AddressBook {
  contacts: {
    phoneNumber: "555 987 6543"
  }
  contacts: {
  }
}
AddressBook {
}123456789
```

![columns_0.png-34.3kB](http://static.zybuluo.com/BrandonLin/s08cdswel3bql0bvv7yvhd0n/columns_0.png)