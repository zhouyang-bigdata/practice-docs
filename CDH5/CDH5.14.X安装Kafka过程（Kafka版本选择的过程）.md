# CDH5.14.X安装Kafka过程（Kafka版本选择的过程）



CDH5.14安装Kafka过程：



在CDH官网中关于Kafka的安装和升级中已经说到，在CDH中，Kafka作为一个分布式的parcel，单独出来作为parcel分发安装包。只要我们把分离开的kafka的服务描述jar包和服务parcel包下载了，就可以实现完美集成了。

注意集成之前请阅读官方文档，特别是版本支持方面。

![img](https://img-blog.csdn.net/20180629112456194?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

查看kafka与CDH版本对应：

https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html#pcm_kafka

![img](https://img-blog.csdn.net/20180629110627311?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

我的CDH是5.14的所以选择3.1的版本。

到：http://archive.cloudera.com/kafka/parcels/latest/

![img](https://img-blog.csdn.net/20180629114427358?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这个网址下载你需要的kafka的parcel版本。我的虚拟机是Centos7的，下载对应的版本，其版本对应如下



EL is short for Red Hat Enterprise Linux (EL).  EL6 is the download for Red Hat 6.x, CentOS 6.x, and CloudLinux 6.x.  EL5 is the download for Red Hat 5.x, CentOS 5.x, CloudLinux 5.x.  EL7 is the download for Red Hat 7.x, CentOS 7.x, and CloudLinux 7.x.  The UNIXy Varnish Plugins run on all the above platforms.

打开manifest.json

![img](https://img-blog.csdn.net/20180629114824183?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

找到这个hash值，将其复制到.sha 文件中去，替换掉原来的hash值。

然后将这三个文件，拷贝到parcel-repo目录下。如果有相同的文件，即manifest.json，只需将之前的重命名即可。

![img](https://img-blog.csdn.net/20180629120341125?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2：下载Kafka-1.2.0.jar

网址：http://archive.cloudera.com/csds/kafka/

上传CSD包KAFKA-1.2.0.jar，到[服务器](https://www.baidu.com/s?wd=%E6%9C%8D%E5%8A%A1%E5%99%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)CDH目录下，路径为/opt/cloudera/csd。

3：![img](https://img-blog.csdn.net/20180629121100406?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

点击继续时候，让你install kafka 的parcel，点进去，检查更新parcel，分配，激活。

![img](https://img-blog.csdn.net/20180629121200884?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180629121252297?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpZ0RhdGFfTWluaW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

配置完成！！