```shell
du -h ##查看当前目录的文件所占的磁盘空间的大小
```

```sh
df -h ##查看磁盘卷的容量大小
```

```shell
systemctl status firewalld ## centos7系统的查看服务的方式
```

```shell
* */1 * * *  sh /home/crontabShell/freeTmp.sh ###crontab -e 编写定时任务的表达式
```

```shell
du --max-depth 1 -lh /var ###递归查看var目录所占用的磁盘空间 
```

```shell
pip install /jupyter/jupyter-1.0.0-py2.py3-none-any.whl --no-index --find-links=/jupyter 离线安装python模块
```

```shell
pip download -d /root/offlinePackage/numpy/ numpy pip 下载离线包
```

```shell
getenforce ## 查看selinux开关状态
```

```sh
##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
setenforce 0 临时关闭selinux 
```

```sh
vi /etc/selinux/config
将SELINUX=enforcing改为SELINUX=disabled  永久关	闭selinux
```

```sh
## mysql增加用户
CREATE USER 'test_admin'@'localhost' IDENTIFIED BY 'admin@123_S';
CREATE USER 'test_admin2'@'%' IDENTIFIED BY '';
```

```sh
## centos7 开放端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

```sh
### mysql 允许root 用户登录
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root';
FLUSH PRIVILEGES;
```

```sh
### 修改mysql用户相关的密码 
mysql -u root
mysql> use mysql;
mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
mysql> FLUSH PRIVILEGES;
```

```sh
一、安装 PostgresSQL
Centos 7 自带的 PostgresSQL 是 9.2 版的。因为，yum 已经做了国内源，速度飞快，所以直接就用 yum 安装了。依次执行以下命令即可，非常简单。
1 sudo yum -y install postgresql-server postgresql
2 sudo service postgresql initdb
3 sudo chkconfig postgresql on
4 sudo systemctl enable postgresql
5 sudo systemctl start postgresql
```

postgresql数据库中dbuser的密码为123456

dbuser用户名的密码也为123456

```sh
##登录postgresql的控制台
sudo -u postgres psql postgres
```

```sh
### 给予非root用户免密sudo
info	ALL=(ALL) 	NOPASSWD:ALL
```

```sh
###设置ulimit参数 
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
* soft memlock -1
* hard memlock -1
```
```shell
cloudera使用自定义的mysql数据库的方法
1、安装mysql数据库
2、为cloudera-manager配置mysql数据库 /usr/share/cmf/schema/scm_prepare_database.sh mysql  -uroot -p123456 scm scm scm
命令的含义为：
	创建一个名叫scm的数据库(第一个scm)，并为这个数据库创建一个用户，用户名为scm(第二个scm)，密码也为scm(第三个scm)
3、删除内嵌数据库的配置项rm -f /etc/cloudera-scm-server/db.mgmt.properties
4、重启service cloudera-scm-server start cloudera-scm-server
```

```sh
###清除系统缓存
sync
echo 3 > /proc/sys/vm/drop_caches
```

```sh
 ##添加软连接  前源 后目标
 ln -s /usr/local/linux/work  /local/linkwork
```

```sh
### 注意一般不要使用root用户做为常用用户 可以配置管理员用户给予root用户的权限
### 具体设置sudoer即可
### 另外一些系统常用目录的介绍
### /var/log /usr/local /var/lib
```
window下查看端口占用的命令组合

```powershell
### 首先查找端口对用的进程号
netstat -aon | findstr "49157"
###然后再查找对应的进程
tasklist | findstr "2720"
```

```shell
###linux 定时关机
shutdown -h 22:00
```

```sh
## 在top命令执行过程中可以使用的一些交互命令。这些命令都是单字母的，如果在命令行中使用了-s选项， 其中一些命令可能会被屏蔽。
h：显示帮助画面，给出一些简短的命令总结说明；
k：终止一个进程；
i：忽略闲置和僵死进程，这是一个开关式命令；
q：退出程序；
r：重新安排一个进程的优先级别；
S：切换到累计模式；
s：改变两次刷新之间的延迟时间（单位为s），如果有小数，就换算成ms。输入0值则系统将不断刷新，默认值是5s；
f或者F：从当前显示中添加或者删除项目；
o或者O：改变显示项目的顺序；
l：切换显示平均负载和启动时间信息；
m：切换显示内存信息；
t：切换显示进程和CPU状态信息；
c：切换显示命令名称和完整命令行；
M：根据驻留内存大小进行排序；
P：根据CPU使用百分比大小进行排序；
T：根据时间/累计时间进行排序；
w：将当前设置写入~/.toprc文件中。
```

```sh
linux下压缩和解压jar包的命令
jar xvf hello.jar   解压hello.jar至当前目录
jar cvf hello.jar hello     利用hello目录创建hello.jar包,并显示创建过程
(8)忽略manifest.mf文件
jar cvfM hello.jar hello    生成的jar包中不包括META-INF目录及manifest.mf文件
 
```