# HDFS 文件操作命令

注意：hdfs dfs 与 hadoop fs 效果一样

常用的就是：

启动hadoop

./sbin/start-dfs.sh

./sbin/start-yarn.sh

或：

./sbin/start-all.sh





hdfs dfs -copyFromLocal /local/data /hdfs/data：将本地文件上传到 hdfs  上（原路径只能是一个文件）

hdfs dfs -moveFromLocal /local/data /hdfs/data：将本地文件移动到 hdfs  上（原路径只能是一个文件）

hdfs dfs -put /tmp/ /hdfs/ ：和 copyFromLocal 区别是，put 原路径可以是文件夹等

hdfs dfs -get /tmp/ /hdfs/ ：local file不能和 hdfs file名字不能相同，否则会提示文件已存在，没有重名的文件会复制到本地

hadoop fs -ls / ：查看根目录文件

hadoop fs -ls /tmp/data：查看/tmp/data目录

hadoop fs -cat /tmp/a.txt ：查看 a.txt，与 -text 一样

hadoop fs -mkdir dir：创建目录dir

hadoop fs -rm -r dir：删除目录dir

hadoop fs -du < hdsf path>  显示hdfs对应路径下每个文件夹和文件的大小

hadoop fs -du - h < hdsf path>  显示hdfs对应路径下每个文件夹和文件的大小,文件的大小用方便阅读的形式表示，例如用64M代替67108864



## HDFS的管理命令

一般管理员才会用下面的命令，举例：

hdfs dfsadmin -report：显示所有dataNode

hdfs dfsadmin -safemode leave：离开安全模式



临时用户

 export HADOOP_USER_NAME=node1