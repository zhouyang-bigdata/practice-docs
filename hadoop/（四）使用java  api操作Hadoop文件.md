# [使用java  api操作Hadoop文件](http://www.cnblogs.com/xuqiang/archive/2011/06/03/2042526.html) 



> 1.概述 
>
> 2.文件操作
>
> ​	2.1  上传本地文件到hadoop  fs
>
> ​	2.2  在hadoop fs中新建文件，并写入
>
> ​	2.3  删除hadoop fs上的文件
>
> ​	2.4  读取文件
>
> 3.目录操作
>
> ​	3.1  在hadoop fs上创建目录
>
> ​	3.2  删除目录
>
> ​	3.3  读取某个目录下的所有文件 
>
> 4.参考资料接代码下载 

### **<1>. 概述** 

hadoop中关于文件操作类基本上全部是在org.apache.hadoop.fs包中，这些api能够支持的操作包含：打开文件，读写文件，删除文件等。

hadoop类库中最终面向用户提供的接口类是FileSystem，该类是个抽象类，只能通过来类的get方法得到具体类。get方法存在几个重载版本，常用的是这个：

static FileSystem get(Configuration conf); 

该类封装了几乎所有的文件操作,例如mkdir，delete等。综上基本上可以得出操作文件的程序库框架：

*operator(){	得到Configuration对象	得到FileSystem对象	进行文件操作}* 

另外需要注意的是，如果想要运行下面的程序的话，需要将程序达成jar包，然后通过hadoop  jar的形式运行，这种方法比较麻烦，另外一种方法就是安装eclipse的hadoop插件，这样能够很多打包的时间。

### **<1>. 文件操作** 

1.1 上传本地文件到文件系统

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     /*
​      \* upload the local file to the hds 
​      \* notice that the path is full like /tmp/test.c
​      */
​     public static void uploadLocalFile2HDFS(String s, String d) 
​         throws IOException
​     {
​         Configuration config = new Configuration();
​         FileSystem hdfs = FileSystem.get(config);
​         
​         Path src = new Path(s);
​         Path dst = new Path(d);
​         
​         hdfs.copyFromLocalFile(src, dst);
​         
​         hdfs.close();
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

1.2 创建新文件，并写入

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     /*
​      \* create a new file in the hdfs.
​     \* notice that the toCreateFilePath is the full path
​     \* and write the content to the hdfs file.
​     */
​     public static void createNewHDFSFile(String toCreateFilePath, String content) throws IOException
​     {
​         Configuration config = new Configuration();
​         FileSystem hdfs = FileSystem.get(config);
​         
​         FSDataOutputStream os = hdfs.create(new Path(toCreateFilePath));

​         os.write(content.getBytes("UTF-8"));
​         
​         os.close();
​         
​         hdfs.close();
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1.3 删除文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    /*
​     \* delete the hdfs file 
​      \* notice that the dst is the full path name
​      */
​     public static boolean deleteHDFSFile(String dst) throws IOException
​     {
​         Configuration config = new Configuration();
​         FileSystem hdfs = FileSystem.get(config);
​         
​         Path path = new Path(dst);
​         boolean isDeleted = hdfs.delete(path);
​         
​         hdfs.close();
​         
​         return isDeleted;
​     }
​     

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1.4  读取文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​     /** read the hdfs file content
​     \* notice that the dst is the full path name
​     */
​     public static byte[] readHDFSFile(String dst) throws Exception
​     {
​         Configuration conf = new Configuration();
​         FileSystem fs = FileSystem.get(conf);
​         
​         // check if the file exists
​        Path path = new Path(dst);
​         if ( fs.exists(path) )
​         {
​             FSDataInputStream is = fs.open(path);
​             // get the file info to create the buffer
​            FileStatus stat = fs.getFileStatus(path);
​             
​             // create the buffer
​            byte[] buffer = new byte[Integer.parseInt(String.valueOf(stat.getLen()))];
​             is.readFully(0, buffer);
​             
​             is.close();
​             fs.close();
​             
​             return buffer;
​         }
​         else
​         {
​             throw new Exception("the file is not found .");
​         }
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### **<2>. 目录操作**

2.1 创建目录

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​      /** make a new dir in the hdfs
​     \* 
​     \* the dir may like '/tmp/testdir'
​     */
​     public static void mkdir(String dir) throws IOException
​     {
​         Configuration conf =  new Configuration();
​         FileSystem fs = FileSystem.get(conf);
​         
​         fs.mkdirs(new Path(dir));
​         
​         fs.close();
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.2 删除目录    

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​      /** delete a dir in the hdfs
​      \* 
​      \* dir may like '/tmp/testdir'
​      */
​     public static void deleteDir(String dir) throws IOException
​     {
​         Configuration conf = new Configuration();
​         FileSystem fs = FileSystem.get(conf);
​         
​         fs.delete(new Path(dir));
​         
​         fs.close();
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.3 读取某个目录下的所有文件 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    public static void listAll(String dir) throws IOException
​     {
​         Configuration conf = new Configuration();
​         FileSystem fs = FileSystem.get(conf);
​         
​         FileStatus[] stats = fs.listStatus(new Path(dir));
​         
​         for(int i = 0; i < stats.length; ++i)
​         {
​             if (stats[i].isFile())
​             {
​                 // regular file
​                System.out.println(stats[i].getPath().toString());
​             }
​             else if (stats[i].isDirectory())
​             {
​                 // dir
​                System.out.println(stats[i].getPath().toString());
​             }
​             else if(stats[i].isSymlink())
​             {
​                 // is s symlink in linux
​                System.out.println(stats[i].getPath().toString());
​             }
​                  
​         }
​         fs.close();
​     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### **<4>. 参考资料及代码下载** 

[/Files/xuqiang/HadoopFSOperations.rar](http://files.cnblogs.com/xuqiang/HadoopFSOperations.rar)

注意的是如果是操作hdfs上文件的话，需要将hadoop-core和common-log的jar包添加到classpath中，在本地运行，如果是mapreduce程序的话，需要将程序打成jar包，之后上传到hdfs上，才能运行。 