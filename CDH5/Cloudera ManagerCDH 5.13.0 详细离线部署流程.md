# Cloudera Manager/CDH 5.13.0 详细离线部署流程

（https://www.jianshu.com/u/a6674389fc08)

 

> 环境约束：
> OS：CentOS 7.0-1511-x86_64
> JDK：jdk1.8.0_152
> Cloudera Manager： centos7-cm5.13.0_x86_64
> CDH：5.13.0-1.cdh5.13.0.p0.29-el7
> Namenode:192.168.134.10/node1
> Datanode:
> 192.168.134.11/node2
> 192.168.134.12/node3
> 
> NTP server:192.168.134.10

> 文中所用软件下载链接：
> [CentOS 7.2 1511 DVD 镜像](https://link.jianshu.com/?t=http%3A%2F%2Farchive.kernel.org%2Fcentos-vault%2F7.2.1511%2Fisos%2Fx86_64%2FCentOS-7-x86_64-Everything-1511.iso)
> [JDK](https://link.jianshu.com/?t=http%3A%2F%2Fwww.oracle.com%2Ftechnetwork%2Fjava%2Fjavase%2Fdownloads%2Fjdk8-downloads-2133151.html)
> [Cloudera Manager tarball](https://link.jianshu.com/?t=http%3A%2F%2Farchive.cloudera.com%2Fcm5%2Fcm%2F5%2Fcloudera-manager-centos7-cm5.13.0_x86_64.tar.gz)
> [CDH parcel](https://link.jianshu.com/?t=http%3A%2F%2Farchive.cloudera.com%2Fcdh5%2Fparcels%2F5.13.0%2FCDH-5.13.0-1.cdh5.13.0.p0.29-el7.parcel)、[sha1 file](https://link.jianshu.com/?t=http%3A%2F%2Farchive.cloudera.com%2Fcdh5%2Fparcels%2F5.13.0%2FCDH-5.13.0-1.cdh5.13.0.p0.29-el7.parcel.sha1)、[manifest.json](https://link.jianshu.com/?t=http%3A%2F%2Farchive.cloudera.com%2Fcdh5%2Fparcels%2F5.13.0%2Fmanifest.json)
> (sha1 file下载后需要改后缀名为.sha)
> [Mysql Server](https://link.jianshu.com/?t=https%3A%2F%2Fdev.mysql.com%2Fget%2FDownloads%2FMySQL-5.7%2Fmysql-community-server-5.7.20-1.el7.x86_64.rpm)、[Mysql Client](https://link.jianshu.com/?t=https%3A%2F%2Fdev.mysql.com%2Fget%2FDownloads%2FMySQL-5.7%2Fmysql-community-client-5.7.20-1.el7.x86_64.rpm)、[Mysql common](https://link.jianshu.com/?t=https%3A%2F%2Fdev.mysql.com%2Fget%2FDownloads%2FMySQL-5.7%2Fmysql-community-common-5.7.20-1.el7.x86_64.rpm)、[Mysql libs](https://link.jianshu.com/?t=https%3A%2F%2Fdev.mysql.com%2Fget%2FDownloads%2FMySQL-5.7%2Fmysql-community-libs-5.7.20-1.el7.x86_64.rpm)
> [JDBC Driver](https://link.jianshu.com/?t=http%3A%2F%2Fcentral.maven.org%2Fmaven2%2Fmysql%2Fmysql-connector-java%2F5.1.44%2Fmysql-connector-java-5.1.44.jar)

------

#### 一、基本环境准备

##### 1. 关闭防火墙和iptables

```
systemctl stop firewalld.service
systemctl stop iptables.service
systemctl disable firewalld.service
systemctl disable iptables.service
```

##### 2. 关闭SELinux

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

##### 3. 配置ntp时间同步服务端

本文配置过程中并未采用集群内的节点作为服务端，实际部署中可以使用集群内的节点作为服务端。<font size="6" color="red">（主节点）</font>

```
vim /etc/ntp.conf
```

打开ntp配置文件，改为以下内容

```
driftfile /var/lib/ntp/drift
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict -6 ::1
restrict 192.168.6.0 mask 255.255.255.0 nomodify notrap
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
server  127.127.1.0  #local clock
fudag   127.127.1.0     stratum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys     
```

或者直接在终端执行：

```
cat << EOF > /etc/ntp.conf
driftfile /var/lib/ntp/drift
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict -6 ::1
restrict 192.168.134.0 mask 255.255.255.0 nomodify notrap
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
server  127.127.1.0  #local clock
fudag   127.127.1.0     stratum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys  
EOF
```

然后启动ntp服务并加入开机自启：

```
service ntpd start
chkconfig ntpd on
```

##### 4. 配置ntp时间同步客户端<font size="6" color="red">（子节点）</font>

打开 **/etc/ntp.conf** 文件，改为我们自己配置好的内容（以192.168.134.10为例）

```
driftfile /var/lib/ntp/drift
restrict 127.0.0.1
restrict ::1
restrict 192.168.134.10 mask 255.255.255.0 nomodify notrap
server 192.168.134.10
server 127.127.1.0
fudge 127.127.1.0 statum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
SYNC_HWCLOCK=yes
```

或者直接在终端执行：

```
cat << EOF > /etc/ntp.conf
driftfile /var/lib/ntp/drift
restrict 127.0.0.1
restrict ::1
restrict 192.168.6.132 mask 255.255.255.0 nomodify notrap
server 192.168.6.132
server 127.127.1.0
fudge 127.127.1.0 statum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
SYNC_HWCLOCK=yes
EOF
```

启动ntp服务并加入开机自启：

```
service ntpd start
chkconfig ntpd on
```

##### 5. 配置主机名

```
cat  << EOF >> /etc/sysconfig/network
NETWORKING=yes
HOSTNAME= node1.myexample.com
EOF
```

##### 6. 在集群<font size="10" color="red">所有主机名</font>规划好之后，修改hosts

```sh
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改eth33 网卡信息IP等内容：

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="3bc2d047-f126-4c5b-a3bc-7bb9a654cb47"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.134.10"
NETMASK="255.255.255.0"
GATEWAY="192.168.134.2"
DNS1="192.168.134.2"
```

```sh
sudo vi /etc/sysconfig/network
```

修改主机名，加入网关：

```
HOSTNAME=node1.myexample.com
```

```
sudo vi /etc/resolv.conf  
```

 配置DNS（与网关相同）

```
nameserver 192.168.134.2
```



```
cat << EOF >> /etc/hosts 
192.168.6.240 node1.myexample.com
192.168.6.241 node101.myexample.com
192.168.6.242 node102.myexample.com
EOF
```

##### 7. 配置无密码登录<font size="10" color="red">（仅主节点）</font>

```
ssh-keygen -t rsa
ssh-copy-id root@node1.myexample.com
ssh-copy-id root@node101.myexample.com
ssh-copy-id root@node102.myexample.com
```

##### 8. 关闭THP服务

在 **/etc/rc.local**文件中设置开机自动关闭THP的语句，因为CentOS7默认开机是不执行rc.local的，所以还要给 **/etc/rc.d/rc.local**可执行权限:

```
cat << EOF >> /etc/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
EOF
chmod +x /etc/rc.d/rc.local
```

##### 9. 安装JDK1.8

建议使用 **/usr/java/jdk1.8** 作为 **JAVA_HOME**,因为YARN等组件默认使用这个目录为 **JAVA_HOME**，直接配置到这里可以避免很多麻烦。
假设jdk的tarball已经拷贝到服务器的 **/usr/java** 目录下 ：

```
tar zxvf jdk-8u152-linux-x64.tar.gz
mkdir jdk1.8
mv jdk1.8.0_152/* jdk1.8/
```

配置环境变量：

```
cat << EOF >> /etc/profile
export JAVA_HOME=/usr/java/jdk1.8
export PATH=\$JAVA_HOME/bin:\$PATH
export CLASSPATH=.:\$JAVA_HOME/lib/dt.jar:\$JAVA_HOME/lib/tools.jar
EOF
source /etc/profile
echo "JAVA_HOME=/usr/java/jdk1.8" >> /etc/environment
```

**完成基本配置后需要重新启动**

#### 二、配置Cloudera Manager Server和Agent

##### 1. 部署文件

把 **cloudera-manager-centos7-cm5.13.0_x86_64.tar.gz**复制到<font size="10" color="red">**所有节点**</font>的 **/opt**目录下解压缩。自动生成 **cloudera、cm-5.13.0**两个文件夹：

```
tar zxvf cloudera-manager-centos7-cm5.13.0_x86_64.tar.gz
```

把JDBC驱动复制到以下目录：

```
cp mysql-connector-java-5.1.44.jar /opt/cm-5.1.3/share/cmf/lib/
cp mysql-connector-java-5.1.44.jar /usr/share/java/
```

##### 2. 安装和配置mysql数据库

首先删除自带的MariaDB：

```
yum erase -y mariadb mariadb-libs
```

安装Mysql，因为依赖关系，这里必须按照这个顺序安装<font color="red">（这里我个人使用的mysql版本是：5.7.20）</font>：

```
rpm -ivh mysql-community-common-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.20-1.el7.x86_64.rpm
```

设置Mysql：

```
systemctl enable mysqld.service
service mysqld start
grep 'temporary password' /var/log/mysqld.log
```

执行完毕之后会有类似如下显示（临时生成的密码会有区别）：

```
2017-12-17T11:26:18.937718Z 1 [Note] A temporary password is generated 
for root@localhost: LgEu(D(<Y9Q?
```

根据上面查找到的密码登录mysql

```sh
su root
mysql -uroot -p
```

以下是mysql命令行：

修改密码，密码为：taima@123ABC，必须包含大小写字母、数字和符号

```
alter user root@localhost identified by 'taima@123ABC';
#授权用户root使用密码passwd从任意主机连接到mysql服务器
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'taima@123ABC' WITH GRANT OPTION;
flush privileges;
```

为ActiveMonitor和Hive创建数据库：

```
create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
quit;
```

创建用户（<font size="10" color="red">所有节点</font>）：

```
useradd --system --home=/opt/cm-5.13.0/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```

##### 3.安装Cloudera Manager Server

为scm建立数据库：

```
/opt/cm-5.13.0/share/cmf/schema/scm_prepare_database.sh mysql -uroot -p scm scm
```

如成功则会出现提示：

```
[main] DbCommandExecutor INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
```

##### 4. 部署CDH的parcel：

将

```
CDH-5.13.0-1.cdh5.13.0.p0.29-el7.parcel
CDH-5.13.0-1.cdh5.13.0.p0.29-el7.parcel.sha
manifest.json
```

三个文件复制到 **/opt/cloudera/parcel-repo/** 目录下。

##### 5. 启动Cloudera Manager Server

# <font size="5" color="red">（node1 启动服务端）</font>执行启动脚本：

```shell
/opt/cm-5.13.0/etc/init.d/cloudera-scm-server start
```

过程非常慢，需要耐心等待5分钟左右，此间可以执行：

```sh
watch netstat -lntp
```

来观察启动情况，如出现**7180**端口的服务启动，则说明启动完成。
执行启动脚本的时候可能会提示：

```
/opt/cm-5.13.0/etc/init.d/cloudera-scm-server:行109: pstree: 未找到命令
```

需要安装psmisc：

```
yum install -y psmisc
```

##### 6. 配置和启动Cloudera Manager Agent

<font size="10" color="red">在每个Agent节点上</font><font size="5" color="red">（node1,node2,node3）</font>修改**config.ini**文件：

```
sed -i 's/server_host=localhost/server_host=node1.myexample.com/g' /opt/cm-5.13.0/etc/cloudera-scm-agent/config.ini
```

<font color="red">主机名"node1.myexample.com"根据实际需求更换成对应的Server's hostname</font>。

<font size="5" color="red">（node1,node2,node3）</font>启动agent：

```shell
/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent start
```

##### 7. 把Cloudera Manager Server/Agent添加到系统服务进行管理

在 **/lib/systemd/system**新建 **cmserver.service**文件,添加系统服务配置(<font size="10" color="red">仅server节点</font>)

```
cat << EOF >> /lib/systemd/system/cmserver.service
[Unit]
Description=Cloudera Manager Server
After=network.target

[Service]
Type=forking
ExecStart=/opt/cm-5.13.0/etc/init.d/cloudera-scm-server start
ExecReload=/opt/cm-5.13.0/etc/init.d/cloudera-scm-server restart
ExecStop=/opt/cm-5.13.0/etc/init.d/cloudera-scm-server stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

新建 **cmagent.service**文件,添加系统服务配置（<font size="10" color="red">所有节点</font>）

```
cat << EOF >> /lib/systemd/system/cmagent.service
[Unit]
Description=Cloudera Manager Agent
After=network.target

[Service]
Type=forking
ExecStart=/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent start
ExecReload=/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent restart
ExecStop=/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

更改配置文件的权限为745（<font color="red" size="10">主节点</font>）：

```
chmod 745 /lib/systemd/system/cmserver.service
chmod 745 /lib/systemd/system/cmagent.service
```

添加到开机自启动（<font color="red" size="10">主节点</font>）：

```
systemctl enable cmserver.service
systemctl enable cmagent.service
```



更改配置文件的权限为745（<font color="red" size="10">子节点</font>）：

```
chmod 745 /lib/systemd/system/cmagent.service
```

添加到开机自启动（<font color="red" size="10">子节点</font>）：

```
systemctl enable cmagent.service
```





然后启动、重启、停止服务就和其它系统服务一样了。



**8.克隆node1的centos系统，逐次生成node2，node3.**





#### 三、部署安装CDH 5.13.0.29

##### 1. 建立集群

用浏览器访问server:7180



![img](https://upload-images.jianshu.io/upload_images/8985600-1ca4d6c88156cf8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

TOC打钩



![img](https://upload-images.jianshu.io/upload_images/8985600-df2b7a00a14576d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

实验环境就选个免费版



![img](https://upload-images.jianshu.io/upload_images/8985600-fde30d20d6d395ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

继续下一步



![img](https://upload-images.jianshu.io/upload_images/8985600-3dce5b22587443e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

如果之前配置都正确的话，就直接选择当前管理的主机。



![img](https://upload-images.jianshu.io/upload_images/8985600-2ea6ee3e6d620ed0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

继续下一步



![img](https://upload-images.jianshu.io/upload_images/8985600-f7d35df7ec2ae56d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

然后就开始较为漫长的安装了



![img](https://upload-images.jianshu.io/upload_images/8985600-8cb2e6ece7433588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

集群安装好之后还要验证一下



![img](https://upload-images.jianshu.io/upload_images/8985600-2cc6b638fc44271a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

这里会将一些检查发现的问题汇总起来，不严重的问题可以先不管，图中提示THP没有关闭是因为我截图的时候忘记把主机的THP关闭了，如果按照基础环境准备的步骤做，是不会提示这个警告的。swapness我也没有管，不过建议搭建的时候如果物理内存足够的话，还是把swapness调小一些，否则真的很卡顿。



![img](https://upload-images.jianshu.io/upload_images/8985600-572661fc5e2ac783.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

然后就可以开始选择安装服务了。



![img](https://upload-images.jianshu.io/upload_images/8985600-9f38af40807ccf3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



##### 2. 安装服务

首先点击“群集”，选择我们装好的cluster（这里我预先装好了HDFS和Zookeeper）



![img](https://upload-images.jianshu.io/upload_images/8985600-c6e8da94f9de9293.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/927/format/webp)

进入cluster控制台，选择添加服务



![img](https://upload-images.jianshu.io/upload_images/8985600-19761d6ab664083f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/735/format/webp)

在这里可以选择安装需要的服务



![img](https://upload-images.jianshu.io/upload_images/8985600-091999a6b68e9eed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



![img](file:///C:\Users\27930\AppData\Local\Temp\ksohtml\wps10A0.tmp.jpg)

HBASE“启用复制”和“启用编制索引”

![img](file:///C:\Users\27930\AppData\Local\Temp\ksohtml\wpsAC06.tmp.jpg)



##### 3. 安装Hive时的注意事项

安装Hive时，除了预先要建立Hive的数据库之外，在安装过程中还会遇到无法连接数据库的错误，此时需要把JDBC的jar包拷贝到 **$HIVE_HOME/lib/** 下面。
在此版本约束下，$HIVE_HOME/lib/一般为：

```
/opt/cloudera/parcels/CDH-5.13.0-1.cdh5.13.0.p0.29/lib/hive/lib/
```

##### 4. <font size="10" color="green">安装失败后，全新重装之前的清理</font>

- 先停止server和agent的服务

```
/opt/cm-5.13.0/etc/init.d/cloudera-scm-server stop
/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent stop
```

- 删除scm等相关的数据库

```
drop database scm;
drop database amon;
drop database hive;
……
```

- 删除agent的部署文件

```
rm -rf /opt/cm-5.13.0/lib/cloudera-scm-agent/*
```

- 删除临时文件

```
rm -rf /tmp/*
```

- 如果之前部署了HDFS服务，还要在每个节点删除dfs的文件

```
rm -rf /dfs/*
```

- 最后重启服务器

#### 四、Trouble shooting

##### 1. 各种状态图不显示，状态图表为灰色小问号，但服务运行状态正常：



![img](https://upload-images.jianshu.io/upload_images/8985600-520dda200b0a445f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

**solution：**
一般这种情况的原因就是运行Host Monitor、Activity Monitor、Service Monitor的主机内存不足，或者这三个服务没有启动，到Cloudera Manager Service控制台进行调整即可。

##### 2. “无法找到主机的 NTP 服务，或该服务未响应时钟偏差请求。”：



![img](https://upload-images.jianshu.io/upload_images/8985600-1ac783b4a0511b69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/755/format/webp)

**solution：**
这种情况的原因是NTP服务没有启动或者配置好，需要手动启动NTP服务，然后手动与服务器进行对时：

```
ntpdate -u ntp.server.com
```

然后重新启动一下agent服务。

```
/opt/cm-5.13.0/etc/init.d/cloudera-scm-agent restart
```

欢迎转载，转载请联系我并注明文章来源。



**3.问题：**

**[taima@node1 software-package]$ sudo rpm -ivh mysql-community-server-5.7.20-1.el7.x86_64.rpm**
**警告：mysql-community-server-5.7.20-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY**
**错误：依赖检测失败：**
	**/usr/bin/perl 被 mysql-community-server-5.7.20-1.el7.x86_64 需要**
	**perl(Getopt::Long) 被 mysql-community-server-5.7.20-1.el7.x86_64 需要**
	**perl(strict) 被 mysql-community-server-5.7.20-1.el7.x86_64 需要**

**solution:**

解决：

```sh
sudo yum install perl
```



**4.问题：**

**mysql -uroot -p AigSX_/b)5sh**

**-bash: 未预期的符号 `)' 附近有语法错误，**



**solution:**

解决：

```sh
mysql -uroot -p
```

enter password:



**5.问题：**

**主机浏览器无法访问虚拟机中centos系统的服务**



**solution:**

解决：
安装firewalld，

```sh
sudo yum -y install firewalld
```



并关闭防火墙



**6.网络相关的文件：**

vi /eth33
vi /etc/hosts
vi /etc/resolve.conf
vi /etc/sysconfig/host

enter password:



**7.问题：**

```
bash: service: command not found
```

**solution：**

解决：

```sh
yum install initscripts -y
```



**8.问题：**

**当前受管**

**假如在安装的时候出现问题，如网络连接中断，机器死机，继续安装的时候可能会出现查询不到机器，并且根据ip搜索机器的时候，出现“当前受管”**

**的状态为“是”，安装失败的机器不能再选择了。**



先停止所有服务。清除数据库。

1> 删除Agent节点的UUID 

      # rm -rf /opt/cm-5.13.0/lib/cloudera-scm-agent/*

2>  清空主节点CM数据库

  进入主节点的Mysql数据库，然后

```
drop database scm;
```

3> 在主节点上重新初始化CM数据库

```sh
/opt/cm-5.13.0/share/cmf/schema/scm_prepare_database.sh mysql -uroot -p scm scm
```

等待一下，连接访问master:7180即可





**9.问题：**

**cloudera中初次启动HDFS，报错：**

**java.io.IOException: NameNode is not formatted**



**solution:**

解决：

```sh
sudo -u hdfs hdfs namenode -format
```



**10.问题：**

**Service has only 0 NodeManager roles running instead of minimum required 1.**

solution:

解决：

看下cm-server，cm-agent 状态：

```sh
/opt/cm-5.13.0/etc/init.d/cloudera-scm-server status
```

如果服务停了，启动一下。



**11.问题：**

**hdfs Permission denied**



**solution:**

```shell
sudo -u hdfs hdfs dfs -mkdir /test-data/
```

