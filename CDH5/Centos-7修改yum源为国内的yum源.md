# [Centos-7修改yum源为国内的yum源](https://www.cnblogs.com/xjh713/p/7458437.html)



 

国外地址yum源下载慢,下到一半就断了,就这个原因就修改它为国内yum源地址

 

国内也就是ali 与 网易

 

以centos7为例 ,以 修改为阿里的yum源

 

1.备份本地yum源

 

```是
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 
```

 

2.获取阿里yum源配置文件

```shell
 wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

 

 

3.更新cache

 

```shell
yum makecache
```

 

 

4.查看

```shell
 yum -y update
```

 

 

5.最后你就可以链接国内镜像了,其实就是那个什么城XXX的 。。。

 

```shell
 mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum makecache

yum -y update 
```

 

[可以借鉴centos修改默认yum源地址  ](http://blog.csdn.net/inslow/article/details/54177191)