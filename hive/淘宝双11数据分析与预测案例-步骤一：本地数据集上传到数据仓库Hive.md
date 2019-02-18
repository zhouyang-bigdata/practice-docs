# 淘宝双11数据分析与预测课程案例-步骤一：本地数据集上传到数据仓库Hive

# 所需知识储备

Linux系统基本命令、Hadoop项目结构、分布式文件系统HDFS概念及其基本原理、数据仓库概念及其基本原理、数据仓库Hive概念及其基本原理

# 技能点

Hadoop的安装与基本操作、HDFS的基本操作、Linux的安装与基本操作、数据仓库Hive的安装与基本操作、基本的数据预处理方法

# 任务清单

1. 安装Linux系统
2. 数据集下载与查看
3. 数据集预处理
4. 把数据集导入分布式文件系统HDFS中
5. 在数据仓库Hive上创建数据库

# Linux系统的安装

本实验全部在Linux系统下开展，因此，必须要安装好Linux系统。

# 实验数据集的下载

本案例采用的数据集压缩包为data_format.zip([点击这里下载data_format.zip数据集](http://pan.baidu.com/s/1cs02Nc))，该数据集压缩包是淘宝2015年双11前6个月(包含双11)的交易数据(交易数据有偏移，但是不影响实验的结果)，里面包含3个文件，分别是用户行为日志文件user_log.csv 、回头客训练集train.csv 、回头客测试集test.csv. 下面列出这3个文件的数据格式定义：

用户行为日志user_log.csv，日志中的字段定义如下：
1. user_id | 买家id
2. item_id | 商品id
3. cat_id | 商品类别id
4. merchant_id | 卖家id
5. brand_id | 品牌id
6. month | 交易时间:月
7. day | 交易事件:日
8. action | 行为,取值范围{0,1,2,3},0表示点击，1表示加入购物车，2表示购买，3表示关注商品
9. age_range | 买家年龄分段：1表示年龄<18,2表示年龄在[18,24]，3表示年龄在[25,29]，4表示年龄在[30,34]，5表示年龄在[35,39]，6表示年龄在[40,49]，7和8表示年龄>=50,0和NULL则表示未知
10. gender | 性别:0表示女性，1表示男性，2和NULL表示未知
11. province| 收获地址省份

回头客训练集train.csv和回头客测试集test.csv，训练集和测试集拥有相同的字段，字段定义如下：

1. user_id | 买家id
2. age_range | 买家年龄分段：1表示年龄<18,2表示年龄在[18,24]，3表示年龄在[25,29]，4表示年龄在[30,34]，5表示年龄在[35,39]，6表示年龄在[40,49]，7和8表示年龄>=50,0和NULL则表示未知
3. gender | 性别:0表示女性，1表示男性，2和NULL表示未知
4. merchant_id | 商家id
5. label | 是否是回头客，0值表示不是回头客，1值表示回头客，-1值表示该用户已经超出我们所需要考虑的预测范围。NULL值只存在测试集，在测试集中表示需要预测的值。

下面，请登录Linux系统（本教程统一采用hadoop用户登录），并在Linux系统中打开浏览器（一般都是火狐Firefox浏览器），然后，在Linux系统的浏览器中打开本网页，[点击这里下载data_format.zip数据集](http://pan.baidu.com/s/1cs02Nc)。如果在下载的时候，你没有修改文件保存路径，火狐浏览器会默认把文件保存在你的当前用户的下载目录下，因为本教程是采用hadoop用户名登录了Linux系统，所以，下载后的文件会被浏览器默认保存到”/home/hadoop/下载/”这目录下面。
现在，请在Linux系统中打开一个终端（可以使用快捷键Ctrl+Alt+T），执行下面命令：

```bash
cd /home/hadoop/下载
ls
```

Shell 命令

通过上面命令，就进入到了data_format.zip文件所在的目录，并且可以看到有个data_format.zip文件。注意，如果你把data_format.zip下载到了其他目录，这里请进入到你自己的存放data_format.zip的目录。
下面需要把data_format.zip进行解压缩，我们需要首先建立一个用于运行本案例的目录dbtaobao，请执行以下命令：

```bash
cd /usr/local
ls
sudo mkdir dbtaobao
//这里会提示你输入当前用户（本教程是hadoop用户名）的密码
//下面给hadoop用户赋予针对dbtaobao目录的各种操作权限
sudo chown -R hadoop:hadoop ./dbtaobao
cd dbtaobao
//下面创建一个dataset目录，用于保存数据集
mkdir dataset
//下面就可以解压缩data_format.zip文件
cd ~  //表示进入hadoop用户的目录
cd 下载
ls
unzip data_format.zip -d /usr/local/dbtaobao/dataset
cd /usr/local/dbtaobao/dataset
ls
```

Shell 命令

现在你就可以看到在dataset目录下有三个文件：test.csv、train.csv、user_log.csv
我们执行下面命令取出user_log.csv前面5条记录看一下
执行如下命令:

```bash
head -5 user_log.csv
```

Shell 命令

可以看到，前5行记录如下：

```
user_id,item_id,cat_id,merchant_id,brand_id,month,day,action,age_range,gender,province
328862,323294,833,2882,2661,08,29,0,0,1,内蒙古
328862,844400,1271,2882,2661,08,29,0,1,1,山西
328862,575153,1271,2882,2661,08,29,0,2,1,山西
328862,996875,1271,2882,2661,08,29,0,1,1,内蒙古
```

# 数据集的预处理

1.删除文件第一行记录，即字段名称
user_log.csv的第一行都是字段名称，我们在文件中的数据导入到数据仓库Hive中时，不需要第一行字段名称，因此，这里在做数据预处理时，删除第一行

```bash
cd /usr/local/dbtaobao/dataset
//下面删除user_log.csv中的第1行
sed -i '1d' user_log.csv //1d表示删除第1行，同理，3d表示删除第3行，nd表示删除第n行
//下面再用head命令去查看文件的前5行记录，就看不到字段名称这一行了
head -5 user_log.csv
```

Shell 命令

2.获取数据集中双11的前100000条数据
由于数据集中交易数据太大，这里只截取数据集中在双11的前10000条交易数据作为小数据集small_user_log.csv
下面我们建立一个脚本文件完成上面截取任务，请把这个脚本文件放在dataset目录下和数据集user_log.csv:

```bash
cd /usr/local/dbtaobao/dataset
vim predeal.sh
```

Shell 命令

上面使用vim编辑器新建了一个predeal.sh脚本文件，请在这个脚本文件中加入下面代码：

```shell
#!/bin/bash
#下面设置输入文件，把用户执行predeal.sh命令时提供的第一个参数作为输入文件名称
infile=$1
#下面设置输出文件，把用户执行predeal.sh命令时提供的第二个参数作为输出文件名称
outfile=$2
#注意！！最后的$infile > $outfile必须跟在}’这两个字符的后面
awk -F "," 'BEGIN{
      id=0;
    }
    {
        if($6==11 && $7==11){
            id=id+1;
            print $1","$2","$3","$4","$5","$6","$7","$8","$9","$10","$11
            if(id==10000){
                exit
            }
        }
    }' $infile > $outfile
```

下面就可以执行predeal.sh脚本文件，截取数据集中在双11的前10000条交易数据作为小数据集small_user_log.csv，命令如下：

```bash
chmod +x ./predeal.sh
./predeal.sh ./user_log.csv ./small_user_log.csv
```

Shell 命令

3.导入数据库
下面要把small_user_log.csv中的数据最终导入到数据仓库Hive中。为了完成这个操作，我们会首先把这个文件上传到分布式文件系统HDFS中，然后，在Hive中创建两个个外部表，完成数据的导入。

a.启动HDFS

下面，请登录Linux系统，打开一个终端，执行下面命令启动Hadoop：

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

b.把user_log.csv上传到HDFS中
现在，我们要把Linux本地文件系统中的user_log.csv上传到分布式文件系统HDFS中，存放在HDFS中的“/dbtaobao/dataset”目录下。
首先，请执行下面命令，在HDFS的根目录下面创建一个新的目录dbtaobao，并在这个目录下创建一个子目录dataset，如下：

```bash
cd /usr/local/hadoop
./bin/hdfs dfs -mkdir -p /dbtaobao/dataset/user_log
```

Shell 命令

然后，把Linux本地文件系统中的small_user_log.csv上传到分布式文件系统HDFS的“/dbtaobao/dataset”目录下，命令如下：

```bash
cd /usr/local/hadoop
./bin/hdfs dfs -put /usr/local/dbtaobao/dataset/small_user_log.csv /dbtaobao/dataset/user_log
```

Shell 命令

下面可以查看一下HDFS中的small_user_log.csv的前10条记录，命令如下：

```bash
cd /usr/local/hadoop
./bin/hdfs dfs -cat /dbtaobao/dataset/user_log/small_user_log.csv | head -10
```



c.在Hive上创建数据库
这里假设你已经完成了Hive的安装，并且使用MySQL数据库保存Hive的元数据。本教程安装的是Hive2.1.0版本，安装目录是“/usr/local/hive”。
下面，请在Linux系统中，再新建一个终端（可以在刚才已经建好的终端界面的左上角，点击“终端”菜单，在弹出的子菜单中选择“新建终端”）。因为需要借助于MySQL保存Hive的元数据，所以，请首先启动MySQL数据库：

```bash
service mysql start  #可以在Linux的任何目录下执行该命令
```



由于Hive是基于Hadoop的数据仓库，使用HiveQL语言撰写的查询语句，最终都会被Hive自动解析成MapReduce任务由Hadoop去具体执行，因此，需要启动Hadoop，然后再启动Hive。由于前面我们已经启动了Hadoop，所以，这里不需要再次启动Hadoop。下面，在这个新的终端中执行下面命令进入Hive：

```bash
cd /usr/local/hive./bin/hive   # 启动Hive
```



启动成功以后，就进入了“hive>”命令提示符状态，可以输入类似SQL语句的HiveQL语句。
下面，我们要在Hive中创建一个数据库dbtaobao，命令如下：

```hive
hive>  create database dbtaobao;
hive>  use dbtaobao;
```



d.创建外部表
关于数据仓库Hive的内部表和外部表的区别，请访问网络文章《[Hive内部表与外部表的区别](http://www.aboutyun.com/thread-7458-1-1.html)》。本教程采用外部表方式。
这里我们要分别在数据库dbtaobao中创建一个外部表user_log，它包含字段（user_id,item_id,cat_id,merchant_id,brand_id,month,day,action,age_range,gender,province）,请在hive命令提示符下输入如下命令：

```hive
hive>  CREATE EXTERNAL TABLE dbtaobao.user_log(user_id INT,item_id INT,cat_id INT,merchant_id INT,brand_id INT,month STRING,day STRING,action INT,age_range INT,gender INT,province STRING) COMMENT 'Welcome to xmu dblab,Now create dbtaobao.user_log!' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION '/dbtaobao/dataset/user_log';
```



e.查询数据
上面已经成功把HDFS中的“/dbtaobao/dataset/user_log”目录下的small_user_log.csv数据加载到了数据仓库Hive中，我们现在可以使用下面命令查询一下：

```hive
hive>  select * from user_log limit 10;
```