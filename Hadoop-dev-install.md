layout: post
title: "Maven搭建Hadoop应用开发环境"
date: 2016-04-26 14:49:51
category: [work]
tags: [hadoop]
---
## 开发环境
>    **Hadoop版本:**2.7.2
>    **Eclipse版本:**Luna 4.4.1

<!--more-->
## 创建maven工程
如果安装了maven可以通过maven命令行创建简单工程,命令为:
```bash
mvn archetype:generate -DgroupId=my.hadoopstudy -DartifactId=hadoopstudy -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
也可以通过eclipse直接创建maven项目
## pom依赖
在pom.xml文件添加Hadoop依赖,内容如下:
```xml
<url>http://maven.apache.org</url>
	<dependencies>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-common</artifactId>
			<version>2.5.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-hdfs</artifactId>
			<version>2.5.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-client</artifactId>
			<version>2.5.1</version>
		</dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
```
这样一个基础的Hadoop工程就构建起来了!
## 实战
为了测试做一些简单的开发.
### HDFS开发
首先测试一下HDFS开发,以之前的伪分布式环境为前提:[http://kiritor.github.io/2016/04/24/Hadoop-install/](http://kiritor.github.io/2016/04/24/Hadoop-install/)
1.启动Hadoop,命令为:
```bash
bin/start-all.sh
```
2.编码
这里通过程序创建文件,列出文件,输出指定文件内容,代码如下:
```java
package com.lcore.hadoop;

import java.io.InputStream;
import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

public class Test {
    public static void main(String[] args) throws Exception {
    	String uri = "hdfs://localhost:9000/";  
        Configuration config = new Configuration();  
        FileSystem fs = FileSystem.get(URI.create(uri), config);  
   
        // 列出hdfs上/user/fkong/目录下的所有文件和目录  
        FileStatus[] statuses = fs.listStatus(new Path("/test/"));  
        for (FileStatus status : statuses) {  
            System.out.println(status);  
        }  
   
        // 在hdfs的/user/fkong目录下创建一个文件，并写入一行文本  
        FSDataOutputStream os = fs.create(new Path("/test/hadoop.log"));  
        os.write("开启我的Hadoop之旅".getBytes());  
        os.flush();  
        os.close();  
   
        // 显示在hdfs的指定文件的内容 
        InputStream is = fs.open(new Path("/test/hadoop.log"));  
        IOUtils.copyBytes(is, System.out, 1024, true); 
	}

}
```
3.运行
直接application或者run on hadoop都可以,必须确保hadoop启动.输出结果如下:
```bash
FileStatus{path=hdfs://localhost:9000/test/hadoop.log; isDirectory=false; length=24; replication=3; blocksize=134217728; modification_time=1461652343934; access_time=1461652342969; owner=lcore; group=supergroup; permission=rw-r--r--; isSymlink=false}
FileStatus{path=hdfs://localhost:9000/test/hexo.md; isDirectory=false; length=1443; replication=1; blocksize=134217728; modification_time=1461565620758; access_time=1461569568851; owner=lcore; group=supergroup; permission=rw-r--r--; isSymlink=false}
FileStatus{path=hdfs://localhost:9000/test/input; isDirectory=true; modification_time=1461652730181; access_time=0; owner=lcore; group=supergroup; permission=rwxr-xr-x; isSymlink=false}
FileStatus{path=hdfs://localhost:9000/test/out; isDirectory=true; modification_time=1461565736828; access_time=0; owner=lcore; group=supergroup; permission=rwxr-xr-x; isSymlink=false}
开启我的Hadoop之旅
```
### MapReduce开发
功能描述:统计出现的名字次数.
1.编码
```java
package com.lcore.hadoop;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;


public class NameCount {
	public static class MyMapper extends Mapper<Object, Text, Text, IntWritable>{  
        private final static IntWritable one = new IntWritable(1);  
        private Text name= new Text();  
   
        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {  
            int idx = value.toString().indexOf(" ");  
            if (idx > 0) {  
                String e = value.toString().substring(0, idx);  
                name.set(e);  
                context.write(name, one);  
            }  
        }  
    }  
   
    public static class MyReducer extends Reducer<Text,IntWritable,Text,IntWritable> {  
        private IntWritable result = new IntWritable();  
   
        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {  
            int sum = 0;  
            for (IntWritable val : values) {  
                sum += val.get();  
            }  
            result.set(sum);  
            context.write(key, result);  
        }  
    }  
   
    public static void main(String[] args) throws Exception {  
        Configuration conf = new Configuration();  
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();  
        if (otherArgs.length < 2) {  
            System.err.println("Usage: NameCount <in> <out>");  
            System.exit(2);  
        }  
        Job job = Job.getInstance(conf, "name count");  
        job.setJarByClass(NameCount.class);  
        job.setMapperClass(MyMapper.class);  
        job.setCombinerClass(MyReducer.class);  
        job.setReducerClass(MyReducer.class);  
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(IntWritable.class);  
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));  
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));  
        System.exit(job.waitForCompletion(true) ? 0 : 1);  
    }  
}  
```
2.通过maven将打成的jar放在hadoop安装目录:share/hadoop/mapreduce/下.
3.创建一下目录及文件用于分析
```bash
event.log.1
event.log.2
event.log.3
```
文件内容格式为:
```bash
Kiritor ...
LCore ...
```
之后复制文件到HDFS上
```bash
bin/hdfs dfs -mkdir /test/input                             #新建目录
bin/hdfs dfs -put ~/Documents/event.log.2 /test/input       #复制文件
bin/hdfs dfs -put ~/Documents/event.log.2 /test/input
bin/hdfs dfs -put ~/Documents/event.log.2 /test/input
```
4.运行
使用如下命令运行:
```bash
bin/hadoop jar share/hadoop/mapreduce/hadoop-0.0.1-SNAPSHOT.jar com.lcore.hadoop.EventCount  /test/input /test/input/out
```
执行结果存放在/test/input/out目录下
5.查看执行结果
使用如下命令查看执行结果:
```bash
bin/hdfs dfs -cat /test/input/out/part-r-00000
#执行结果
KK	2
Kiritor	3
L.Tao	4
LCore	5
```
over!
