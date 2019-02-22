# win10 单机运行spark-shell命令部署

##### 一 下载spark-1.6.0-bin-hadoop2.6.tgz 安装包，解压



##### 二 配置环境变量

**在windows下面配置环境变量(新建SPARK_HOME系统变量，输入spark安装文件路径，在PATH中加入%SPARK_HOME%\bin;变量即可。)**

​       进入windows控制台中，直接输入spark-shell即可显示如下图。

​     否则，需要进入下载的spark1.6.0的下载安装目录中，执行spark-shell

​       执行结果如下：

![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326195836299-370931511.png)

**三 测试是否正确**

​    **3.1.准备数据**

　　E:\scala\spark\testdata中的work.txt文件中写入以下文件

  　　  Hello you

​    　　Hello me

**3.2.输入并查看结果输出**

![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326195902705-893098391.png)

![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326195910236-1852114505.png)

![img](https://images2015.cnblogs.com/blog/735738/201703/735738-20170326195920627-1582693762.png)