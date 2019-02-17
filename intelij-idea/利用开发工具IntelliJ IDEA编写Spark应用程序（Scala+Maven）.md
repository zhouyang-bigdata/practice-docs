# 利用开发工具IntelliJ IDEA编写Spark应用程序（Scala+Maven）


对Scala代码进行打包编译时，可以采用Maven，也可以采用sbt，相对而言，业界更多使用sbt。这里介绍IntelliJ IDEA和Maven的组合使用方法。

## 1.安装IntelliJ IDEA

在idea官网下载idea。

## 2.在Intellij里安装scala插件，并配置JDK，scala SDK

首先如图打开plugins界面。
![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/2.png)
然后我们点击Install JetBrain Plugins..如下图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/3.png)
搜索并安装scala。
等待安装完成后我们就可以配置JDK跟scala SDK。

### 配置JDK

首先我们打开Project Structure，如下图。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/4.png)
然后我们添加JDK（这里默认已经安装JDK并且配置了环境变量），操作按下面两张图。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/5.png)![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/6.png)

### 配置全局Scala SDK

还是在Project Structure界面，操作如下。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/7.png)![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/8.png)
然后我们右键我们添加的SDK选择Copy to Project Libraries…OK确认。如图
![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/25.png)
配置好后我们就可以创建工程文件了。

## 3.创建Maven工程文件

我们点击初始界面的Create New Project进入如图界面。并按图创建Maven工程文件。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/9.png)
然后我们要填写GroupId：dblab；以及ArtifactId：WordCount，如图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/10.png)
然后按下图填写各项，这一步容易出错请认真填写。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/11.png)
到此创建工程文件完成。

## 4.前置的一些配置

### 将scala框架添加到项目

在IDEA启动后进入的界面中，可以看到界面左侧的项目界面，已经有一个名称为WordCount的工程。请在该工程名称上右键单击，在弹出的菜单中，选择Add Framework Surport ，在左侧有一排可勾选项，找到scala，勾选即可。

### 创建WordCount文件夹，并作为suorces root

在src文件夹下创建一个WordCount文件夹。
右键新建的文件夹，按图把该文件夹设置为sources root。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/12.png)

## 5.两次的代码黏贴

### 黏贴wordcount代码到WordCount.scala

然后就可以通过右键刚刚设置为sources root的wordcount文件夹，就有了new->scala class的选项。
我们新建一个scala class，并且命名WordCount，选着为object类型。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/13.png)
我们打开建好的WordCount.scala文件，清空！然后黏贴以下代码：

```
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

object WordCount {
  def main(args: Array[String]) {
    val inputFile =  "file:///usr/local/spark/mycode/wordcount/word.txt"
    val conf = new SparkConf().setAppName("WordCount").setMaster("local")
    val sc = new SparkContext(conf)
    val textFile = sc.textFile(inputFile)
    val wordCount = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
    wordCount.foreach(println)
  }
}
```

### 黏贴pom.xml代码

现在清空pom.xml，把以下代码黏贴到pom.xml里。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>dblab</groupId>
    <artifactId>WordCount</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spark.version>2.1.0</spark.version>
        <scala.version>2.11</scala.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-mllib_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>

        </plugins>
    </build>

</project>
```

黏贴好后，我们右键点击工程文件夹，更新一下，按下图操作。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/14.png)
这时候我们注意要记得点击右下角的“Import Changes Enables Auto-Import ”中的”Enables Auto-Import”。如图。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/15.png)
这时候我们需要等待一段时间，可以看底部的进度条。等执行完毕，我们再进行后面的操作。

## 6.运行WordCount程序

在WordCount.scala代码窗口内的任意位置，我们右键点击，可以唤出菜单，选择Run ‘WordCount’。运行的结果如下。注意根据代码，你必须有/usr/local/spark/mycode/wordcount/word.txt这个文件。输出信息较多，你可以拖动一下寻找结果信息。
![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/16.png)

## 7.打包WordCount程序的jar包

首先打开File->Project Structure。如图。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/17-1.png)
然后选择Artifacts->绿色加号->Jar->From moduleswith dependencies…如图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/18.png)
选择Main Class，如图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/19.png)
然后因为我们只是在Spark上运行的，所以我们要删除下图红框里多余的部分，保留WordCount.jar以及‘WordCount’ compile output。小提示，这里可以利用Ctrl+A全选功能，选中全部选项，然后，配合Crtl+鼠标左键进行反选，也就是按住Ctrl键的同时用鼠标左键分别点击WordCount.jar和‘WordCount’ compile output，从而不选中这两项，最后，点击页面中的删除按钮（是一个减号图标），这样就把其他选项都删除，只保留了WordCount.jar以及‘WordCount’ compile output。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/20.png)
然后我们点击Apply，再点击OK，如图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/21.png)
接着我们就可以导出Jar包了。选择Build->Build Artifacts…，在弹出的窗口选择Bulid就可以了。如下图：![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/22.png)
导出的Jar包会在工程文件“/home/wordcount/”目录下的“out/artifacts/WordCount_jar”目录下。我们把他复制到/home/hadoop目录下。也就是主文件夹目录下，如下图![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/23.png)
实际上，可以用命令来复制WordCount.jar文件，请打开一个Linux终端，输入如下命令：

```bash
cd ~cp /home/hadoop/WordCount/out/artifacts/WordCount_jar/WordCount.jar /home/hadoop
```

Shell 命令

然后我们在终端执行以下命令，运行Jar包：

```bash
cd ~/usr/local/spark/bin/spark-submit --class WordCount /home/hadoop/WordCount.jar
```

Shell 命令

运行结果如下（输出的信息较多请上下翻一下就能找到），要求还是跟上述一样要有那个文件存在。![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/02/24.png)
到此我们就完成了本次的任务了。