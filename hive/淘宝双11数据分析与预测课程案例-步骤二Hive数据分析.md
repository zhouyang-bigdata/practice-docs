# 淘宝双11数据分析与预测案例-步骤二:Hive数据分析

# 所需知识储备

数据仓库Hive概念及其基本原理、SQL语句、数据库查询分析

# 技能点

数据仓库Hive基本操作、创建数据库和表、使用SQL语句进行查询分析

# 任务清单

1.启动Hadoop和Hive
2.创建数据库和表
3.简单查询分析
4.查询条数统计分析
5.关键字条件查询分析
6.根据用户行为分析
7.用户实时查询分析

# 一、操作Hive

请登录Linux系统（本教程统一采用hadoop用户名登录系统），然后，打开一个终端（可以按快捷键Ctrl+Alt+T）。
本教程中，Hadoop的安装目录是“/usr/local/hadoop”，Hive的安装目录是“/usr/local/hive”。
因为需要借助于MySQL保存Hive的元数据，所以，请首先启动MySQL数据库，请在终端中输入下面命令：

```bash
service mysql start  # 可以在Linux的任何目录下执行该命令
```

Shell 命令

由于Hive是基于Hadoop的数据仓库，使用HiveQL语言撰写的查询语句，最终都会被Hive自动解析成MapReduce任务由Hadoop去具体执行，因此，需要启动Hadoop，然后再启动Hive。

请执行下面命令启动Hadoop（如果你已经启动了Hadoop就不用再次启动了）：

```bash
cd /usr/local/hadoop
./sbin/start-dfs.sh
```

Shell 命令

然后，执行jps命令看一下当前运行的进程：

```bash
jps
```

Shell 命令

如果出现下面这些进程，说明Hadoop启动成功了。

```
3765 NodeManager
3639 ResourceManager
3800 Jps
3261 DataNode
3134 NameNode
3471 SecondaryNameNode
```

下面，继续执行下面命令启动进入Hive：

```bash
cd /usr/local/hive
./bin/hive   //启动Hive
```

Shell 命令

通过上述过程，我们就完成了MySQL、Hadoop和Hive三者的启动。
启动成功以后，就进入了“hive>”命令提示符状态，可以输入类似SQL语句的HiveQL语句。

然后，在“hive>”命令提示符状态下执行下面命令：

```hive
hive> use dbtaobao; -- 使用dbtaobao数据库
hive> show tables; -- 显示数据库中所有表。
hive> show create table user_log; -- 查看user_log表的各种属性；
```



执行结果如下：

```
OK
CREATE EXTERNAL TABLE `user_log`(
  `user_id` int,
  `item_id` int,
  `cat_id` int,
  `merchant_id` int,
  `brand_id` int,
  `month` string,
  `day` string,
  `action` int,
  `age_range` int,
  `gender` int,
  `province` string)
COMMENT 'Welcome to xmu dblab,Now create dbtaobao.user_log!'
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  'field.delim'=',',
  'serialization.format'=',')
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://localhost:9000/dbtaobao/dataset/user_log'
TBLPROPERTIES (
  'numFiles'='1',
  'totalSize'='4729522',
  'transient_lastDdlTime'='1487902650')
Time taken: 0.084 seconds, Fetched: 28 row(s)
```

可以执行下面命令查看表的简单结构：

```hive
hive> desc user_log;
```

hive

执行结果如下：

```
OK
user_id                 int
item_id                 int
cat_id                  int
merchant_id             int
brand_id                int
month                   string
day                     string
action                  int
age_range               int
gender                  int
province                string
Time taken: 0.029 seconds, Fetched: 11 row(s)
```

二、简单查询分析
先测试一下简单的指令：

```hive
hive> select brand_id from user_log limit 10; -- 查看日志前10个交易日志的商品品牌
```

hive

执行结果如下：

```
OK
5476
5476
6109
5476
5476
5476
5476
5476
5476
6300
```

如果要查出每位用户购买商品时的多种信息，输出语句格式为 select 列1，列2，….，列n from 表名；
比如我们现在查询前20个交易日志中购买商品时的时间和商品的种类

```hive
hive> select month,day,cat_id from user_log limit 20;
```

hive

执行结果如下：

```
OK
11  11  1280
11  11  1280
11  11  1181
11  11  1280
11  11  1280
11  11  1280
11  11  1280
11  11  1280
11  11  1280
11  11  962
11  11  81
11  11  1432
11  11  389
11  11  1208
11  11  1611
11  11  420
11  11  1611
11  11  1432
11  11  389
11  11  1432
```

有时我们在表中查询可以利用嵌套语句，如果列名太复杂可以设置该列的别名，以简化我们操作的难度，以下我们可以举个例子：

```hive
hive> select ul.at, ul.ci  from (select action as at, cat_id as ci from user_log) as ul limit 20;
```



执行结果如下：

```
OK
0   1280
0   1280
0   1181
2   1280
0   1280
0   1280
0   1280
0   1280
0   1280
0   962
2   81
2   1432
0   389
2   1208
0   1611
0   420
0   1611
0   1432
0   389
0   1432
```

这里简单的做个讲解，action as at ,cat_id as ci就是把action 设置别名 at ,cat_id 设置别名 ci，FROM的括号里的内容我们也设置了别名ul，这样调用时用ul.at,ul.ci,可以简化代码。

三、查询条数统计分析
经过简单的查询后我们同样也可以在select后加入更多的条件对表进行查询,下面可以用函数来查找我们想要的内容。
(1)用聚合函数count()计算出表内有多少条行数据

```hive
hive> select count(*) from user_log; -- 用聚合函数count()计算出表内有多少条行数据
```



执行结果如下：

```bash
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = hadoop_20170224103108_d6361e99-e76a-43e6-94b5-3fb0397e3ca6
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:31:10,085 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local792612260_0001
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 954982 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
10000
Time taken: 1.585 seconds, Fetched: 1 row(s)
```

Shell 命令

我们可以看到，得出的结果为OK下的那个数字54925330（
(2)在函数内部加上distinct，查出uid不重复的数据有多少条
下面继续执行操作：

```hive
hive> select count(distinct user_id) from user_log; -- 在函数内部加上distinct，查出user_id不重复的数据有多少条
```

hive

执行结果如下：

```
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = hadoop_20170224103141_47682fd4-132b-4401-813a-0ed88f0fb01f
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:31:42,501 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1198900757_0002
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 1901772 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
358
Time taken: 1.283 seconds, Fetched: 1 row(s)
```

(3)查询不重复的数据有多少条(为了排除客户刷单情况) **

```hive
hive> select count(*) from (select user_id,item_id,cat_id,merchant_id,brand_id,month,day,action from user_log group by user_id,item_id,cat_id,merchant_id,brand_id,month,day,action having count(*)=1)a;
```



执行结果如下：

```
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = hadoop_20170224103334_3391e361-c710-4162-b022-2658f41fc228
Total jobs = 2
Launching Job 1 out of 2
Number of reduce tasks not specified. Estimated from input data size: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:33:35,890 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1670662918_0003
Launching Job 2 out of 2
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:33:37,026 Stage-2 map = 100%,  reduce = 100%
Ended Job = job_local2041177199_0004
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 2848562 HDFS Write: 0 SUCCESS
Stage-Stage-2:  HDFS Read: 2848562 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
4754
Time taken: 2.478 seconds, Fetched: 1 row(s)
```

可以看出，排除掉重复信息以后，只有4754条记录。
注意：嵌套语句最好取别名，就是上面的a，否则很容易出现如下错误.
![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/QQ%E6%88%AA%E5%9B%BE20161115160822.png)

四．关键字条件查询分析
1.以关键字的存在区间为条件的查询
使用where可以缩小查询分析的范围和精确度，下面用实例来测试一下。
(1)查询双11那天有多少人购买了商品

```hive
hive> select count(distinct user_id) from user_log where action='2';
```

执行结果如下：

```
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = hadoop_20170224103500_44e669ed-af51-4856-8963-002d85112f32
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:35:01,940 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1951453719_0005
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 3795352 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
358
Time taken: 1.231 seconds, Fetched: 1 row(s)
```

2.关键字赋予给定值为条件，对其他数据进行分析
取给定时间和给定品牌，求当天购买的此品牌商品的数量

```hive
hive> select count(*) from user_log where action='2' and brand_id=2661;
```

执行结果如下：

```
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = hadoop_20170224103541_4640ca81-1d25-48f8-8d9d-6027f2befdb9
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2017-02-24 10:35:42,457 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1749705881_0006
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 4742142 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
3
Time taken: 1.258 seconds, Fetched: 1 row(s)
```

五．根据用户行为分析
从现在开始，我们只给出查询语句，将不再给出执行结果。
1．查询一件商品在某天的购买比例或浏览比例

```hive
hive> select count(distinct user_id) from user_log where action='2'; -- 查询有多少用户在双11购买了商品
```



```hive
hive> select count(distinct user_id) from user_log; -- 查询有多少用户在双11点击了该店
```



根据上面语句得到购买数量和点击数量，两个数相除即可得出当天该商品的购买率。
2.查询双11那天，男女买家购买商品的比例

```hive
hive> select count(*) from user_log where gender=0; --查询双11那天女性购买商品的数量hive> select count(*) from user_log where gender=1; --查询双11那天男性购买商品的数量
```



上面两条语句的结果相除，就得到了要要求的比例。
3.给定购买商品的数量范围，查询某一天在该网站的购买该数量商品的用户id

```hive
hive> select user_id from user_log where action='2' group by user_id having count(action='2')>5; -- 查询某一天在该网站购买商品超过5次的用户id
```

hive

六.用户实时查询分析
不同的品牌的浏览次数

```hive
hive> create table scan(brand_id INT,scan INT) COMMENT 'This is the search of bigdatataobao' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE; -- 创建新的数据表进行存储
hive> insert overwrite table scan select brand_id,count(action) from user_log where action='2' group by brand_id; --导入数据
hive> select * from scan; -- 显示结果
```