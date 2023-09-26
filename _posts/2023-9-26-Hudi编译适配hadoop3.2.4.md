---
layout:     post
title:      Hudi编译适配hadoop3.2.4
subtitle:   Hudi适配hadoop3.x
date:       2023-9-26
author:     zhmingyong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 大数据
    - 数据湖
    - Hudi
    - Hadoop 3.2.4
---


# Hudi编译适配hadoop3.2.4


本文讲解`hudi`如何编译适配Hadoop3.x，其中hudi采用版本0.12.3，hadoop采用版本3.2.4。

## 1.默认配置

Hudi的编译配置文件为<HUDI\_HOME>/pom.xml（其中<HUDI\_HOME>是Hudi根目录），默认配置的hadoop版本为2.10.1。相关配置如下图：

![](https://zhmingyong.github.io/img/hudi-hadoop3.x_1.png)

编译参数-Dhadoop：表示选择hadoop2.10.1；

## 2.编译方法

适配Hadoop3.x的hudi编译有两种方案。（本示例中采用方案二）

**一、通过编译命令指定**

```shell
mvn clean package -DskipTests -Dspark3.2 -Dflink1.14 -Dscala-2.12 -Dhadoop.version=3.2.4
```

**二、修改配置文件**

修改<HUDI\_HOME>/pom.xml中hadoop.version为3.2.4

![](https://zhmingyong.github.io/img/hudi-hadoop3.x_2.png)

然后，通过如下命令编译适配hadoop：

```shell
mvn clean package -DskipTests -Drat.skip=true -Dhadoop
```

编译成功有如下提示：

```shell
[INFO] Reactor Summary for Hudi 0.12.3:
[INFO] 
[INFO] Hudi ............................................... SUCCESS [  1.010 s]
[INFO] hudi-tests-common .................................. SUCCESS [  0.585 s]
[INFO] hudi-common ........................................ SUCCESS [  8.311 s]
[INFO] hudi-hadoop-mr ..................................... SUCCESS [  2.003 s]
[INFO] hudi-sync-common ................................... SUCCESS [  0.659 s]
[INFO] hudi-hive-sync ..................................... SUCCESS [  2.212 s]
[INFO] hudi-aws ........................................... SUCCESS [  1.127 s]
[INFO] hudi-timeline-service .............................. SUCCESS [  0.838 s]
[INFO] hudi-client ........................................ SUCCESS [  0.028 s]
[INFO] hudi-client-common ................................. SUCCESS [  5.412 s]
[INFO] hudi-spark-client .................................. SUCCESS [ 10.706 s]
[INFO] hudi-spark-datasource .............................. SUCCESS [  0.025 s]
[INFO] hudi-spark-common_2.12 ............................. SUCCESS [ 11.686 s]
[INFO] hudi-spark3-common ................................. SUCCESS [  4.374 s]
[INFO] hudi-spark3.2plus-common ........................... SUCCESS [  5.399 s]
[INFO] hudi-spark3.2.x_2.12 ............................... SUCCESS [  9.605 s]
[INFO] hudi-java-client ................................... SUCCESS [  1.307 s]
[INFO] hudi-spark_2.12 .................................... SUCCESS [ 18.723 s]
[INFO] hudi-utilities_2.12 ................................ SUCCESS [  3.523 s]
[INFO] hudi-utilities-bundle_2.12 ......................... SUCCESS [ 18.907 s]
[INFO] hudi-cli ........................................... SUCCESS [  8.284 s]
[INFO] hudi-flink-client .................................. SUCCESS [ 11.176 s]
[INFO] hudi-gcp ........................................... SUCCESS [  0.608 s]
[INFO] hudi-datahub-sync .................................. SUCCESS [  0.520 s]
[INFO] hudi-adb-sync ...................................... SUCCESS [  0.802 s]
[INFO] hudi-sync .......................................... SUCCESS [  0.024 s]
[INFO] hudi-hadoop-mr-bundle .............................. SUCCESS [  8.850 s]
[INFO] hudi-datahub-sync-bundle ........................... SUCCESS [ 10.978 s]
[INFO] hudi-hive-sync-bundle .............................. SUCCESS [  8.952 s]
[INFO] hudi-aws-bundle .................................... SUCCESS [ 10.142 s]
[INFO] hudi-gcp-bundle .................................... SUCCESS [  8.953 s]
[INFO] hudi-spark3.2-bundle_2.12 .......................... SUCCESS [ 17.265 s]
[INFO] hudi-presto-bundle ................................. SUCCESS [ 10.007 s]
[INFO] hudi-utilities-slim-bundle_2.12 .................... SUCCESS [ 15.409 s]
[INFO] hudi-timeline-server-bundle ........................ SUCCESS [ 13.625 s]
[INFO] hudi-trino-bundle .................................. SUCCESS [  9.932 s]
[INFO] hudi-examples ...................................... SUCCESS [  0.027 s]
[INFO] hudi-examples-common ............................... SUCCESS [  1.421 s]
[INFO] hudi-examples-spark ................................ SUCCESS [  4.631 s]
[INFO] hudi-flink-datasource .............................. SUCCESS [  0.025 s]
[INFO] hudi-flink1.14.x ................................... SUCCESS [  0.715 s]
[INFO] hudi-flink ......................................... SUCCESS [ 40.273 s]
[INFO] hudi-examples-flink ................................ SUCCESS [  0.945 s]
[INFO] hudi-examples-java ................................. SUCCESS [  1.715 s]
[INFO] hudi-flink1.13.x ................................... SUCCESS [  0.758 s]
[INFO] hudi-flink1.15.x ................................... SUCCESS [  0.770 s]
[INFO] hudi-kafka-connect ................................. SUCCESS [  3.661 s]
[INFO] hudi-flink1.14-bundle .............................. SUCCESS [ 16.274 s]
[INFO] hudi-kafka-connect-bundle .......................... SUCCESS [ 24.778 s]
[INFO] hudi-spark2_2.12 ................................... SUCCESS [  8.170 s]
[INFO] hudi-spark2-common ................................. SUCCESS [  0.034 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  05:46 min
[INFO] Finished at: 2023-09-14T11:54:44+08:00
[INFO] ------------------------------------------------------------------------
```

一定要出现“BUILD SUCCESS“提示。

然后执行hudi-cli.sh：

```shell
# ./hudi-cli/hudi-cli.sh Client jar location not set, please set it in conf/hudi-env.sh……Welcome to Apache Hudi CLI. Please type help if you are looking for help. hudi->
```

## 3.问题

HoodieParquetDataBlock找不到合适的`ByteArrayOutputStream`构造器

**一、问题描述**

Hudi0.12.0编译配适hadoop 3.2.4版本时，编译异常：FSDataOutputStream(java.io.ByteArrayOutputStream), 找不到合适的构造器。错误日志如下：

```shell
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.0:compile (default-compile) on project hudi-common: Compilation failure[ERROR] /D:/Workspace/Apache/apache-hudi/hudi-common/src/main/java/org/apache/hudi/common/table/log/block/HoodieParquetDataBlock.java:[112,44] 对于FSDataOutputStream(java.io.ByteArrayOutputStream), [ERROR] org.apache.hadoop.fs.FSDataOutputStream.FSDataOutputStream(java.io.OutputStream,org.apache.hadoop.fs.FileSystem.Statistics)[ERROR] org.apache.hadoop.fs.FSDataOutputStream.FSDataOutputStream(java.io.OutputStream,org.apache.hadoop.fs.FileSystem.Statistics,long)[ERROR]org.apache.hadoop.fs.FSDataOutputStream.FSDataOutputStream(java.io.OutputStream,org.apache.hadoop.fs.FileSystem.Statistics,long)[ERROR]
```

**二、问题分析**

​问题代码如下：

```java
    try (FSDataOutputStream outputStream = new FSDataOutputStream(baos)) {      try (HoodieParquetStreamWriter<IndexedRecord> parquetWriter = new HoodieParquetStreamWriter<>(outputStream, avroParquetConfig)) {        for (IndexedRecord record : records) {          String recordKey = getRecordKey(record).orElse(null);          parquetWriter.writeAvro(recordKey, record);        }        outputStream.flush();      }    }
```

查看FSDataOutputStream的代码，可以看到FSDataOutputStream只有2个构造器了。新的构造器加入了Statistics stats和long startPosition两个参数，用于进行输出流的io统计。

```java
  public FSDataOutputStream(OutputStream out, Statistics stats) {    this(out, stats, 0L);  }   public FSDataOutputStream(OutputStream out, Statistics stats, long startPosition) {    super(new FSDataOutputStream.PositionCache(out, stats, startPosition));    this.wrappedStream = out;  }
```

需要激活统计的话，至少需要传入Statistics。Statistics有两个构造函数，一个是传入系统的schema,一个是传入另一个Statistics对象。第二个构造器是不可能了，那第一个可以吗？在hudi的HoodieParquetDataBlock可以拿到系统的schema吗？

```java
    public Statistics(String scheme) {      this.scheme = scheme;      this.rootData = new FileSystem.Statistics.StatisticsData();      this.threadData = new ThreadLocal();      this.allData = new HashSet();    }     public Statistics(FileSystem.Statistics other) {      this.scheme = other.scheme;      this.rootData = new FileSystem.Statistics.StatisticsData();      other.visitAll(new FileSystem.Statistics.StatisticsAggregator<Void>() {        public void accept(FileSystem.Statistics.StatisticsData data) {          Statistics.this.rootData.add(data);        }         public Void aggregate() {          return null;        }      });      this.threadData = new ThreadLocal();      this.allData = new HashSet();    }
```

发现拿不到，所以只能传空。修改代码文件./hudi-common/src/main/java/org/apache/hudi/common/table/log/block/HoodieParquetDataBlock.java。

![](https://zhmingyong.github.io/img/hudi-hadoop3.x_3.png)

查看hudi的其他代码，传空是普遍现象哦。重新编译后，顺利编译通过，可以放心使用。