---
layout:     post
title:      ApacheHudi+MinIO+HMS构建现代数据湖（转载）
subtitle:   Hudi+MinIO+HMS构建数据湖
date:       2024-04-15
author:     zhmingyong
header-img: img/hudi/hudi-20240415.png
catalog: true
tags:
    - Hudi
    - MinIO
	  - HMS
---

原文链接： https://mp.weixin.qq.com/s/rCJ6XikkU03VvuLj3eFoGQ


Apache Hudi 已成为管理现代数据湖的领先开放表格式之一，直接在现代数据湖中提供核心仓库和数据库功能。这在很大程度上是由于 Hudi 提供了高级功能，例如表、事务、更新插入/删除、高级索引、流式摄取服务、数据集群/压缩优化和并发控制。

我们已经探索了[1] MinIO 和 Hudi 如何协同工作来构建现代数据湖。这篇博文旨在以这些知识为基础，提供一种利用 Hive Metastore 服务 (HMS[2]) 的 Hudi 和 MinIO 的替代实现。部分源于 Hadoop 生态系统的起源故事，Hudi 的许多大规模数据实现仍然利用 HMS。通常从遗留系统的迁移故事涉及某种程度的混合，因为要利用所涉及的所有产品中最好的产品来取得成功。

## Hudi 与 MinIO：成功的组合
Hudi 从依赖 HDFS 到像 MinIO 这样的云原生对象存储的演变，与数据行业从单一且不合适的遗留解决方案的转变完美契合。MinIO 的性能[3]、可扩展性[4]和成本效益[5]使其成为存储和管理 Hudi 数据的理想选择。此外，Hudi 对现代数据中的 Apache Spark、Flink、Presto、Trino、StarRocks 等的优化与 MinIO 无缝集成，以实现大规模的云原生性能。这种兼容性代表了现代数据湖架构中的一个重要模式。

![](https://zhmingyong.github.io/img/hudi-20240415/hudi+minio.png)
## HMS集成：增强数据治理和管理
虽然 Hudi 提供开箱即用的核心数据管理功能，但与 HMS 集成增加了另一层控制和可见性。以下是 HMS 集成如何使大规模 Hudi 部署受益：

• 改进的数据治理：HMS 集中元数据管理，在整个数据湖中实现一致的访问控制、沿袭跟踪和审计。这可确保数据质量、合规性并简化治理流程。

• 简化的架构管理：在 HMS 中定义和实施 Hudi 表的架构，确保跨管道和应用程序的数据一致性和兼容性。HMS 模式演化功能允许在不破坏管道的情况下适应不断变化的数据结构。

• 增强的可见性和发现性：HMS 为所有数据资产（包括 Hudi 表）提供中央目录。这有助于分析师和数据科学家轻松发现和探索数据。

## 入门：满足先决条件
要完成本教程需要设置一些软件。以下是详细信息：

• Docker 引擎：这个强大的工具允许您在称为容器的标准化软件单元中打包和运行应用程序。

• Docker Compose：充当协调器，简化多容器应用程序的管理。它有助于轻松定义和运行复杂的应用程序。

安装：Docker Desktop 安装程序提供了一个方便的一站式解决方案，用于在特定平台（Windows、macOS 或 Linux）上安装 Docker 和 Docker Compose。这通常比单独下载和安装它们更容易。安装 Docker Desktop 或 Docker 和 Docker Compose 的组合后可以通过在终端中运行以下命令来验证它们的存在：

```bash
docker-compose --version
```
请注意，本教程是针对 linux/amd64 构建的，为了使其适用于 Mac M2 芯片，还需要安装 Rosetta 2。可以通过运行以下命令在终端窗口中执行此操作：

```bash
softwareupdate --install-rosetta
```
在 Docker Desktop 设置中还需要启用 Rosetta 在 Apple Silicone 上进行 x86_64/amd64 二进制模拟。为此，请导航至“设置”→“常规”，然后选中“Rosetta”框，如下所示。

![](https://zhmingyong.github.io/img/hudi-20240415/docker_rosetta.png)

## 在MinIO上集成HMS和Hudi
本教程使用 StarRocks 的演示存储库。克隆此处[6]找到的存储库。在终端窗口中导航到 documentation-samples 目录，然后导航到 hudi 文件夹并运行以下命令：

```bash
docker compose up
```
运行上述命令后应该会看到 StarRocks、HMS 和 MinIO 已启动并正在运行。通过 http://localhost:9000/ 访问 MinIO 控制台并使用凭据 admin:password 登录，即可看到存储桶 warehouse 已自动创建。

![](https://zhmingyong.github.io/img/hudi-20240415/minio_buckets.png)

## 使用 Spark Scala 插入数据
运行以下命令来访问 spark-hudi 容器内的 shell：

```bash
docker exec -it hudi-spark-hudi-1 /bin/bash
```
然后运行以下命令将进入 Spark REPL：

```bash
/spark-3.2.1-bin-hadoop3.2/bin/spark-shell
```
进入 shell 后执行以下 Scala 行来创建数据库、表并向该表中插入数据：

```bash
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._
import org.apache.spark.sql.Row
import org.apache.spark.sql.SaveMode._
import org.apache.hudi.DataSourceReadOptions._
import org.apache.hudi.DataSourceWriteOptions._
import org.apache.hudi.config.HoodieWriteConfig._
import scala.collection.JavaConversions._

val schema = StructType(Array(
  StructField("language", StringType, true),
  StructField("users", StringType, true),
  StructField("id", StringType, true)
))

val rowData= Seq(
  Row("Java", "20000", "a"),
  Row("Python", "100000", "b"),
  Row("Scala", "3000", "c")
)

val df = spark.createDataFrame(rowData, schema)

val databaseName = "hudi_sample"
val tableName = "hudi_coders_hive"
val basePath = "s3a://warehouse/hudi_coders"

df.write.format("hudi").
option(org.apache.hudi.config.HoodieWriteConfig.TABLE_NAME, tableName).
option(RECORDKEY_FIELD_OPT_KEY, "id").
option(PARTITIONPATH_FIELD_OPT_KEY, "language").
option(PRECOMBINE_FIELD_OPT_KEY, "users").
option("hoodie.datasource.write.hive_style_partitioning", "true").
option("hoodie.datasource.hive_sync.enable", "true").
option("hoodie.datasource.hive_sync.mode", "hms").
option("hoodie.datasource.hive_sync.database", databaseName).
option("hoodie.datasource.hive_sync.table", tableName).
option("hoodie.datasource.hive_sync.partition_fields", "language").
option("hoodie.datasource.hive_sync.partition_extractor_class", "org.apache.hudi.hive.MultiPartKeysValueExtractor").
option("hoodie.datasource.hive_sync.metastore.uris", "thrift://hive-metastore:9083").
mode(Overwrite).
save(basePath)
```
现在已经使用 Hudi 和 HMS 设置了 MinIO 数据湖。导航回 http://localhost:9000/ 以查看仓库文件夹已填充。

![](https://zhmingyong.github.io/img/hudi-20240415/minio_browser.png)

## 数据探索
可以选择在同一 Shell 中利用以下 Scala 来进一步探索数据。

```bash
val hudiDF = spark.read.format("hudi").load(basePath + "/*/*")

hudiDF.show()

val languageUserCount = hudiDF.groupBy("language").agg(sum("users").as("total_users"))
languageUserCount.show()

val uniqueLanguages = hudiDF.select("language").distinct()
uniqueLanguages.show()

// Stop the Spark session
System.exit(0)
```

## 构建云原生现代数据湖
Hudi、MinIO 和 HMS 无缝协作，为构建和管理大规模现代数据湖提供全面的解决方案。通过集成这些技术可以获得释放数据全部潜力所需的敏捷性、可扩展性和安全性。

### 引用链接

[1] 已经探索了: [https://blog.min.io/streaming-data-lakes-hudi-minio/?utm_term=&utm_campaign=&utm_source=adwords&utm_medium=ppc&hsa_acc=8976569894&hsa_cam=20466868543&hsa_grp=&hsa_ad=&hsa_src=x&hsa_tgt=&hsa_kw=&hsa_mt=&hsa_net=adwords&hsa_ver=3&gad_source=1&gclid=CjwKCAiA0PuuBhBsEiwAS7fsNQanvkO02hS9l_MY0HTiH2XjcPVJVDITKXq4ydcHFmDYUGrwgVH0WBoCXtEQAvD_BwE](https://blog.min.io/streaming-data-lakes-hudi-minio/?utm_term=&utm_campaign=&utm_source=adwords&utm_medium=ppc&hsa_acc=8976569894&hsa_cam=20466868543&hsa_grp=&hsa_ad=&hsa_src=x&hsa_tgt=&hsa_kw=&hsa_mt=&hsa_net=adwords&hsa_ver=3&gad_source=1&gclid=CjwKCAiA0PuuBhBsEiwAS7fsNQanvkO02hS9l_MY0HTiH2XjcPVJVDITKXq4ydcHFmDYUGrwgVH0WBoCXtEQAvD_BwE)

[2] HMS: [https://hive.apache.org/?ref=blog.min.io](https://hive.apache.org/?ref=blog.min.io)

[3] 性能: [https://blog.min.io/nvme_benchmark/](https://blog.min.io/nvme_benchmark/)

[4] 可扩展性: [https://min.io/product/scalable-object-storage?ref=blog.min.io](https://min.io/product/scalable-object-storage?ref=blog.min.io)

[5] 成本效益: [https://blog.min.io/the-long-term-costs-of-storage-in-the-cloud/](https://blog.min.io/the-long-term-costs-of-storage-in-the-cloud/)

[6] 此处: [https://github.com/StarRocks/demo?ref=blog.min.io](https://github.com/StarRocks/demo?ref=blog.min.io)