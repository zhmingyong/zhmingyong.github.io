---
layout:     post
title:      TDengine 入门教程——数据库SQL操作 | 建库、建表、数据读写
subtitle:   TDengine 建库、建表、数据读写
date:       2023-09-5
author:     zhmingyong
header-img: img/post-bg-hadoop.jpeg
catalog: true
tags:
    - TDengine
    - 数据库SQL操作
---


# TDengine 入门教程——数据库SQL操作 | 建库、建表、数据读写



#### 文章目录

- 一、前文
- 二、库操作

- 2.1 创建库
- 2.2 删除库
- 2.3 查询库

- 三、表操作

- 3.1 创建表
- 3.2 删除表
- 3.3 查询所有表
- 3.4 查询表结构

- 四、数据写入

- 4.1 插入数据
- 4.2 修改标签值

- 五、数据查询

- 5.1 查询总表
- 5.2 查询子表

- 六、参考



## 一、前文

> TDengine 入门教程——导读

## 二、库操作

### 2.1 创建库

> CREATE DATABASE test;

- `BUFFER`: 一个 VNODE 写入内存池大小，单位为 MB，默认为 96，最小为 3，最大为 16384。
- `CACHEMODEL`：表示是否在内存中缓存子表的最近数据。默认为 none。

- none：表示不缓存。
- last_row：表示缓存子表最近一行数据。这将显著改善 LAST_ROW 函数的性能表现。
- last_value：表示缓存子表每一列的最近的非 NULL 值。这将显著改善无特殊影响（WHERE、ORDER BY、GROUP BY、INTERVAL）下的 LAST 函数的性能表现。
- both：表示同时打开缓存最近行和列功能。

- `CACHESIZE`：表示每个 vnode 中用于缓存子表最近数据的内存大小。默认为 1 ，范围是[1, 65536]，单位是 MB。
- `COMP`：表示数据库文件压缩标志位，缺省值为 2，取值范围为 [0, 2]。

- 0：表示不压缩。
- 1：表示一阶段压缩。
- 2：表示两阶段压缩。

- `DURATION`：数据文件存储数据的时间跨度。可以使用加单位的表示形式，如 DURATION 100h、DURATION 10d 等，支持 m（分钟）、h（小时）和 d（天）三个单位。不加时间单位时默认单位为天，如 DURATION 50 表示 50 天。
- `WAL_FSYNC_PERIOD`：当 WAL 参数设置为 2 时，落盘的周期。默认为 3000，单位毫秒。最小为 0，表示每次写入立即落盘；最大为 180000，即三分钟。
- `MAXROWS`：文件块中记录的最大条数，默认为 4096 条。
- `MINROWS`：文件块中记录的最小条数，默认为 100 条。
- `KEEP`：表示数据文件保存的天数，缺省值为 3650，取值范围 [1, 365000]，且必须大于或等于 DURATION 参数值。数据库会自动删除保存时间超过 KEEP 值的数据。KEEP 可以使用加单位的表示形式，如 KEEP 100h、KEEP 10d 等，支持 m（分钟）、h（小时）和 d（天）三个单位。也可以不写单位，如 KEEP 50，此时默认单位为天。
- `PAGES`：一个 VNODE 中元数据存储引擎的缓存页个数，默认为 256，最小 64。一个 VNODE 元数据存储占用 PAGESIZE * PAGES，默认情况下为 1MB 内存。
- `PAGESIZE`：一个 VNODE 中元数据存储引擎的页大小，单位为 KB，默认为 4 KB。范围为 1 到 16384，即 1 KB 到 16 MB。
- `PRECISION`：数据库的时间戳精度。ms 表示毫秒，us 表示微秒，ns 表示纳秒，默认 ms 毫秒。
- `REPLICA`：表示数据库副本数，取值为 1 或 3，默认为 1。在集群中使用，副本数必须小于或等于 DNODE 的数目。
- `RETENTIONS`：表示数据的聚合周期和保存时长，如 RETENTIONS 15s:7d,1m:21d,15m:50d 表示数据原始采集周期为 15 秒，原始数据保存 7 天；按 1 分钟聚合的数据保存 21 天；按 15 分钟聚合的数据保存 50 天。目前支持且只支持三级存储周期。
- `STRICT`：表示数据同步的一致性要求，默认为 off。

- on 表示强一致，即运行标准的 raft 协议，半数提交返回成功。
- off 表示弱一致，本地提交即返回成功。

- `WAL_LEVEL`：WAL 级别，默认为 1。

- 1：写 WAL，但不执行 fsync。
- 2：写 WAL，而且执行 fsync。

- `VGROUPS`：数据库中初始 vgroup 的数目。
- `SINGLE_STABLE`：表示此数据库中是否只可以创建一个超级表，用于超级表列非常多的情况。

- 0：表示可以创建多张超级表。
- 1：表示只可以创建一张超级表。

- `WAL_RETENTION_PERIOD`：wal 文件的额外保留策略，用于数据订阅。wal 的保存时长，单位为 s。单副本默认为 0，即落盘后立即删除。-1 表示不删除。多副本默认为 4 天。
- `WAL_RETENTION_SIZE`：wal 文件的额外保留策略，用于数据订阅。wal 的保存的最大上限，单位为 KB。单副本默认为 0，即落盘后立即删除。多副本默认为-1，表示不删除。
- `WAL_ROLL_PERIOD`：wal 文件切换时长，单位为 s。当 wal 文件创建并写入后，经过该时间，会自动创建一个新的 wal 文件。单副本默认为 0，即仅在落盘时创建新文件。多副本默认为 1 天。
- `WAL_SEGMENT_SIZE`：wal 单个文件大小，单位为 KB。当前写入文件大小超过上限后会自动创建一个新的 wal 文件。默认为 0，即仅在落盘时创建新文件。

```plain
taos> CREATE DATABASE test;
Query OK, 0 of 0 rows affected (0.042633s)1.2.
taos> CREATE DATABASE test1;

DB error: Out of dnodes (0.000307s)1.2.3.
```

### 2.2 删除库

> DROP DATABASE test;

```plain
taos> DROP DATABASE test;
Query OK, 0 of 0 rows affected (0.024260s)1.2.
```

### 2.3 查询库

> SHOW DATABASES;

```plain
taos> SHOW DATABASES;
              name              |
=================================
 information_schema             |
 performance_schema             |
 test                           |
Query OK, 3 rows in database (0.003425s)1.2.3.4.5.6.7.
```

## 三、表操作

### 3.1 创建表

> CREATE TABLE test.weather(ts timestamp, temperature float) tags(location nchar(64));

```plain
taos> CREATE TABLE test.weather(ts timestamp, temperature float) tags(location nchar(64));
Query OK, 0 of 0 rows affected (0.001351s)1.2.
```

### 3.2 删除表

> DROP TABLE test.weather;

```plain
taos> DROP TABLE test.weather;
Query OK, 0 of 0 rows affected (0.001149s)1.2.
```

### 3.3 查询所有表

> SHOW TABLES;

```plain
taos> show tables from test;

DB error: syntax error near "from test;" (0.000089s)
taos> show tables;

DB error: db not specified (0.000047s)
taos> USE test;
Database changed.

taos> SHOW TABLES;
           table_name           |
=================================
 t1                             |
 d1                             |
Query OK, 2 rows in database (0.003124s)1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.
```

### 3.4 查询表结构

> DESCRIBE test.weather;

```plain
taos> DESCRIBE test.weather;
             field              |         type         |   length    |   note   |
=================================================================================
 ts                             | TIMESTAMP            |           8 |          |
 temperature                    | FLOAT                |           4 |          |
 location                       | NCHAR                |          64 | TAG      |
Query OK, 3 rows in database (0.000595s)1.2.3.4.5.6.7.
```

## 四、数据写入

### 4.1 插入数据

> INSERT INTO t1 USING test.weather tags(‘北京’) values(now, 18.2);

- 可以看出，t1的tags不会变，固定了
- t1和d1都是子表

```plain
taos> INSERT INTO t1 USING test.weather tags('北京') values(now, 18.2);
Query OK, 1 of 1 rows affected (0.000994s)

taos> INSERT INTO t1 USING test.weather tags('北京') values(now, 17.2);
Query OK, 1 of 1 rows affected (0.006636s)

taos> INSERT INTO t1 USING test.weather tags('上海') values(now, 16.2);
Query OK, 1 of 1 rows affected (0.000858s)

taos> SELECT * from test.weather;
           ts            |     temperature      |            location            |
==================================================================================
 2022-09-09 23:00:54.526 |             18.20000 | 北京                           |
 2022-09-09 23:01:37.493 |             17.20000 | 北京                           |
 2022-09-09 23:01:58.623 |             16.20000 | 北京                           |
Query OK, 3 rows in database (0.003118s)1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.
```

### 4.2 修改标签值

> ALTER TABLE t1 set tag location=‘上海’;

```plain
taos> USE test;
Database changed.

taos> ALTER TABLE t1 SET TAG location='上海';
Query OK, 0 of 0 rows affected (0.000942s)

taos> SELECT ts,temperature,location FROM t1;
           ts            |     temperature      |            location            |
==================================================================================
 2022-09-09 23:00:54.526 |             18.20000 | 上海                           |
 2022-09-09 23:01:37.493 |             17.20000 | 上海                           |
 2022-09-09 23:01:58.623 |             16.20000 | 上海                           |
Query OK, 3 rows in database (0.001336s)1.2.3.4.5.6.7.8.9.10.11.12.13.
```

## 五、数据查询

### 5.1 查询总表

> SELECT * from test.weather;

```plain
taos> SELECT * FROM test.weather ;
           ts            |     temperature      |            location            |
==================================================================================
 2022-09-09 23:12:54.634 |              1.70000 | 厦门                           |
 2022-09-09 23:00:54.526 |             18.20000 | 北京                           |
 2022-09-09 23:01:37.493 |             17.20000 | 北京                           |
 2022-09-09 23:01:58.623 |             16.20000 | 北京                           |
Query OK, 4 rows in database (0.002965s)1.2.3.4.5.6.7.8.
```

### 5.2 查询子表

```plain
taos> SELECT * from t1;
           ts            |     temperature      |
=================================================
 2022-09-09 23:00:54.526 |             18.20000 |
 2022-09-09 23:01:37.493 |             17.20000 |
 2022-09-09 23:01:58.623 |             16.20000 |
Query OK, 3 rows in database (0.003236s)1.2.3.4.5.6.7.
taos> SELECT ts,temperature,location from t1;
           ts            |     temperature      |            location            |
==================================================================================
 2022-09-09 23:00:54.526 |             18.20000 | 北京                           |
 2022-09-09 23:01:37.493 |             17.20000 | 北京                           |
 2022-09-09 23:01:58.623 |             16.20000 | 北京                           |
Query OK, 3 rows in database (0.006667s)1.2.3.4.5.6.7.
taos> SELECT ts,temperature,location from d1;
           ts            |     temperature      |            location            |
==================================================================================
 2022-09-09 23:12:54.634 |              1.70000 | 厦门                           |
Query OK, 1 rows in database (0.003230s)1.2.3.4.5.
```

## 六、参考

[ 数据库 | TDengine 文档](https://docs.taosdata.com/taos-sql/database/)

[ 查询数据 | TDengine 文档](