layout: post
title: Mac下Hadoop2.7.x配置伪分布环境(wordcount运行)
category: work
date: 2016-04-28 23:34:07
tags: [hadoop]
---
关于Hadoop的安装与配置其实挺多了,不过大多是1.x.x系列的.将自己的安装配置做个笔记记录.
## 前言
>    操作系统:Mac OS X 10.10
>    Hadoop版本: 2.7.2

安装JDK并配置环境变量
## 配置Mac OS自身ssh环境
配置ssh环境,在终端中输入
```bash
ssh localhost
```
<!--more-->
如果出现错误提示信息,表示当前用户没有权限.更改设置如下:进入system prference-->sharing-->勾选remote login,并设置allow access for all users.再次输入"ssh localhost"输入密码确认,即可看到ssh成功.
比较麻烦的是,每次都会要求输入用户密码.Hadoop提供了无密码验证登陆的方式:
>    1. 创建ssh-key,命令:ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
>       ssh-keygen表示生成秘钥:-t表示秘钥类型;-P用于提供密语;-f指定生成的秘钥文件.这个命令在"~/.ssh"文件夹下创建两个文件
>       id_dsa喝id_das_pub,是ssh的一对私钥和公钥
>    2. 将公钥和私钥追加到授权的key中,输入:cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

然后就可以无密码验证登陆了,成功后输入如下信息:
```bash
LCore:~ lcore$ ssh localhost
Password:
Last login: Sun Apr 24 22:06:11 2016
LCore:~ lcore$ 
```
## 下载Hadoop
下载Hadoop安装包有两种方式:
1、直接官网下载
[http://mirrors.hust.edu.cn/apache/hadoop/core/stable/hadoop-2.7.1.tar.gz]http://mirrors.hust.edu.cn/apache/hadoop/core/stable/hadoop-2.7.2.tar.gz
2、使用shell进行下载,命令如下
```bash
wget http://mirrors.hust.edu.cn/apache/hadoop/core/stable/hadoop-2.7.2.tar.gz
```
接下来解压缩Hadoop到事先指定好的目录,移动并压缩:
```bash
mv ~/Downloads/hadoop-2.7.2.tar.gz ~/dev/hadoop
tar -zxvf hadoop-2.7.2.tar.gz 
```
## 设置环境变量
在实际配置Hadoop之前,我们需要配置好环境变量
```bash
export HADOOP_HOME=/Users/lcore/dev/hadoop/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
```
Tips:export设置只对当前的bash登陆session有效.可是使用source命令使其立即生效.
## Hadoop配置
2.7.x系列的版本配置文件在hadoop/etc/hadoop目录下
### 配置hadoop-env.sh
如果该文件里没有配置JDK,可以使用如下命令配置:
```bash
export JAVA_HOME=${JAVA_HOME}
```
否则会报错找不到JDK
### 配置core-site.xml
该配置文件用于指定NameNode的主机名与端口
```bash
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/Users/lcore/dev/hadoop/hadoop-2.7.2/tmp</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131702</value>
    </property>
</configuration>
```
### 配置hdfs-site.xml
该配置文件制定了HDFS的默认参数及副本数,因为仅运行在一个节点上,所以这里的副本数为1
```bash
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/Users/lcore/dev/hadoop/hadoop-2.7.2/tmp/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/Users/lcore/dev/hadoop/hadoop-2.7.2/tmp/hdfs/data</value>
    </property>

    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>localhost:9001</value>
    </property>
    <property>
      <name>dfs.webhdfs.enabled</name>
      <value>true</value>
    </property>
</configuration>
```
其中dfs.namenodename.dir和dfs.datanode.data.dir的路径可以自由设置,最好在hadoop.tmp.dir目录下.
### 配置mapred-site.xml.template
该配置文件制定了JobTracker的主机名与端口
```bash
<configuration>

	<property>

	<name>mapred.job.tracker</name>

	<value>localhost:9001</value>

	</property>

</configuration>
```
## 运行hadoop
在配置完成之后,运行Hadoop
### 初始化HDFS
在hadoop主目录下输入一下命令:
```bash
bin/hdfs namenode -format
```
过程中需要进行ssh验证,因为之前已经登录了,所以初始化过程键入y即可,输出结果如下:
<center>![hdfs](/img/hadoop-hdfs.png)</center>
表示已经初始化完成
### 开启NameNode和DataNode守护进程
使用如下命令开启：
```bash
sbin/start-dfs.sh
```
成功的截图如下:
<center>![name](/img/hadoop-namenode.png)</center>
### 查看进程信息
使用jps命令查看进程出现如下信息表示DataNode和NameNode都已经开启
```bash
4368 Jps
3202 NameNode
3411 SecondaryNameNode
3295 DataNode
```
### 查看Web UI
在浏览器中输入:[http://localhost:50070](http://localhost:50070),即可查看相关信息,截图如下:
<center>![webui](/img/hadoop-webui.png)</center>
此时,hadoop伪分布式集成环境已经搭建完成,接下来运行一下WordCount例子.
## 运行WordCount
1.新建一个测试文件,内容随意(这里我把一篇博客文章做测试)
2.在HDFS中新建测试文件夹test.命令为:bin/hdfs dfs -mkdir /test,使用"bin/hdfs dfs -ls"命令查看是否新建成功.
3.上传测试文件,命令如下:bin/hdfs dfs -put ~/dev/blog/hexo.md /test/,可使用之前命令查看是否上传成功.
4.运行wordcount,命令如下:bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /test/hexo.md /test/out,运行完成后,在/test目录下生成名为out的目录,其内容如下所示:
```bash
-rw-r--r--   1 lcore supergroup          0 2016-04-25 14:28 /test/out/_SUCCESS
-rw-r--r--   1 lcore supergroup       1470 2016-04-25 14:28 /test/out/part-r-00000
```
表示已经运行成功,结果存放在part-r-00000文件中
5.使用命令:bin/hadoop fs -cat /test/out/part-r-00000 查看结果为如下图所示:
<center>![wordcount](/img/hadoop-wordcount.png)</center>
到此,hadoop-2.7.2伪分布式配置到此结束了!
