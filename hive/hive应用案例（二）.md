# hive应用案例（二）

## 场景：统计每日用户登陆总数。

每分钟的原始日志内容如下：<http://www.blue.com/uid=xxxxxx&ip=xxxxxx>

假设只有两个字段,uid和ip,其中uid是用户的uid，是用户的唯一标识，ip是用户的登陆ip，每日的记录行数是10亿，要统计出一天用户登陆的总数。

##   Hive建表

hive>CREATE TABLElogin (uid  STRING, ip  STRING) 

PARTITIONEDBY (dt STRING)

ROW FORMATDELIMITED

FIELDSTERMINATED BY ','

STORED ASTEXTFILE;

表名是login,字段之间以,隔开,存储是TEXT,其次还以dt这个字段作为分区。创建成功之后,在hdfs上新建了/user/hive/warehouse/login目录。

##   格式化原始日志

将每天的每分钟的原始日志，转换成以下文件格式

123,17.6.2.6

112,11.3.6.2

………..

 根据文件大小，合并文件，例如合并为24个文件。

##   数据入库到hive

hive>LOADDATA LOCAL  INPATH'/data/login/20120713/*' OVERWRITE INTO TABLE login PARTITION (dt='20120713');

执行成功，转换过的文件会上传到hdfs的/user/hive/warehouse/login/dt=20120713目录里。

##   在hive上统计分析

hive>selectcount(distinct uid) from login where dt=’20120713’;

使用dt这个分区条件查询，就可以避免hive去查询其他分区的文件，减少IO操作，这个是hive分区很重要的特性，也是以天为单位，作为login表分区的重要意义。

执行完毕后，就可以在命令里出现结果，一般通过管道执行hive shell命令，读取管道的内容，把结果入库到mysql里就完成分析。