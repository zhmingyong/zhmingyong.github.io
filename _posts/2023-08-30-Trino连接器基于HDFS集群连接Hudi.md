---
layout:     post
title:      Trino连机器基于HDFS集群连接Hudi
subtitle:   namenode高可用下使用hudi连接器
date:       2023-08-30
author:     zhmingyong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Trino
    - Hudi
    - namenode高可用
---

## trino连接hudi最小配置示例(hudi.properties)
```shell
connector.name=hudi
hive.metastore.uri=thrift://example.net:9083
```
namenode单节点情况下，hudi写入任务配置确定活动namenode节点的情况下，trino读取无异常；namenode集群情况下，hudi写入任务配置集群名称并写入hudi后，trino读取异常

## 集群配置异常
### 配置
mycluster为集群名称
```shell
connector.name=hudi
hive.metastore.uri=thrift://example.net:9083
```

### 异常
```shell
org.jkiss.dbeaver.model.sql.DBSQLException: SQL 错误 [65536]: Query failed (#20230830_095654_00017_pkr2x): Failed to get instance of org.apache.hadoop.fs.FileSystem
	at org.jkiss.dbeaver.model.impl.jdbc.exec.JDBCStatementImpl.executeStatement(JDBCStatementImpl.java:133)
	at org.jkiss.dbeaver.model.impl.jdbc.struct.JDBCTable.readData(JDBCTable.java:187)
	at org.jkiss.dbeaver.ui.controls.resultset.ResultSetJobDataRead.lambda$0(ResultSetJobDataRead.java:123)
	at org.jkiss.dbeaver.model.exec.DBExecUtils.tryExecuteRecover(DBExecUtils.java:189)
	at org.jkiss.dbeaver.ui.controls.resultset.ResultSetJobDataRead.run(ResultSetJobDataRead.java:121)
	at org.jkiss.dbeaver.ui.controls.resultset.ResultSetViewer$ResultSetDataPumpJob.run(ResultSetViewer.java:5101)
	at org.jkiss.dbeaver.model.runtime.AbstractJob.run(AbstractJob.java:105)
	at org.eclipse.core.internal.jobs.Worker.run(Worker.java:63)
Caused by: java.sql.SQLException: Query failed (#20230830_095654_00017_pkr2x): Failed to get instance of org.apache.hadoop.fs.FileSystem
	at io.trino.jdbc.AbstractTrinoResultSet.resultsException(AbstractTrinoResultSet.java:1937)
	at io.trino.jdbc.TrinoResultSet.getColumns(TrinoResultSet.java:318)
	at io.trino.jdbc.TrinoResultSet.create(TrinoResultSet.java:61)
	at io.trino.jdbc.TrinoStatement.internalExecute(TrinoStatement.java:262)
	at io.trino.jdbc.TrinoStatement.execute(TrinoStatement.java:240)
	at org.jkiss.dbeaver.model.impl.jdbc.exec.JDBCStatementImpl.execute(JDBCStatementImpl.java:330)
	at org.jkiss.dbeaver.model.impl.jdbc.exec.JDBCStatementImpl.executeStatement(JDBCStatementImpl.java:131)
	... 7 more
Caused by: org.apache.hudi.exception.HoodieIOException: Failed to get instance of org.apache.hadoop.fs.FileSystem
	at org.apache.hudi.common.fs.FSUtils.getFs(FSUtils.java:111)
	at org.apache.hudi.common.fs.FSUtils.getFs(FSUtils.java:102)
	at io.trino.plugin.hudi.HudiMetadata.isHudiTable(HudiMetadata.java:202)
	at io.trino.plugin.hudi.HudiMetadata.getTableHandle(HudiMetadata.java:107)
	at io.trino.plugin.hudi.HudiMetadata.getTableHandle(HudiMetadata.java:73)
	at io.trino.spi.connector.ConnectorMetadata.getTableHandle(ConnectorMetadata.java:121)
	at io.trino.plugin.base.classloader.ClassLoaderSafeConnectorMetadata.getTableHandle(ClassLoaderSafeConnectorMetadata.java:1063)
	at io.trino.metadata.MetadataManager.lambda$getTableHandle$5(MetadataManager.java:282)
	at java.base/java.util.Optional.flatMap(Optional.java:289)
	at io.trino.metadata.MetadataManager.getTableHandle(MetadataManager.java:276)
	at io.trino.metadata.MetadataManager.getRedirectionAwareTableHandle(MetadataManager.java:1546)
	at io.trino.metadata.MetadataManager.getRedirectionAwareTableHandle(MetadataManager.java:1538)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.getTableHandle(StatementAnalyzer.java:5374)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitTable(StatementAnalyzer.java:2179)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitTable(StatementAnalyzer.java:479)
	at io.trino.sql.tree.Table.accept(Table.java:60)
	at io.trino.sql.tree.AstVisitor.process(AstVisitor.java:27)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.process(StatementAnalyzer.java:496)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.analyzeFrom(StatementAnalyzer.java:4439)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitQuerySpecification(StatementAnalyzer.java:2906)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitQuerySpecification(StatementAnalyzer.java:479)
	at io.trino.sql.tree.QuerySpecification.accept(QuerySpecification.java:155)
	at io.trino.sql.tree.AstVisitor.process(AstVisitor.java:27)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.process(StatementAnalyzer.java:496)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.process(StatementAnalyzer.java:504)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitQuery(StatementAnalyzer.java:1470)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.visitQuery(StatementAnalyzer.java:479)
	at io.trino.sql.tree.Query.accept(Query.java:107)
	at io.trino.sql.tree.AstVisitor.process(AstVisitor.java:27)
	at io.trino.sql.analyzer.StatementAnalyzer$Visitor.process(StatementAnalyzer.java:496)
	at io.trino.sql.analyzer.StatementAnalyzer.analyze(StatementAnalyzer.java:458)
	at io.trino.sql.analyzer.Analyzer.analyze(Analyzer.java:79)
	at io.trino.sql.analyzer.Analyzer.analyze(Analyzer.java:71)
	at io.trino.execution.SqlQueryExecution.analyze(SqlQueryExecution.java:258)
	at io.trino.execution.SqlQueryExecution.<init>(SqlQueryExecution.java:196)
	at io.trino.execution.SqlQueryExecution$SqlQueryExecutionFactory.createQueryExecution(SqlQueryExecution.java:818)
	at io.trino.dispatcher.LocalDispatchQueryFactory.lambda$createDispatchQuery$0(LocalDispatchQueryFactory.java:149)
	at io.trino.$gen.Trino_410____20230830_095517_2.call(Unknown Source)
	at com.google.common.util.concurrent.TrustedListenableFutureTask$TrustedFutureInterruptibleTask.runInterruptibly(TrustedListenableFutureTask.java:131)
	at com.google.common.util.concurrent.InterruptibleTask.run(InterruptibleTask.java:74)
	at com.google.common.util.concurrent.TrustedListenableFutureTask.run(TrustedListenableFutureTask.java:82)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)
Caused by: java.net.UnknownHostException: mycluster
	at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:447)
	at org.apache.hadoop.hdfs.NameNodeProxiesClient.createProxyWithClientProtocol(NameNodeProxiesClient.java:131)
	at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:355)
	at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:289)
	at org.apache.hadoop.hdfs.DistributedFileSystem.initialize(DistributedFileSystem.java:176)
	at io.trino.hdfs.TrinoFileSystemCache.createFileSystem(TrinoFileSystemCache.java:159)
	at io.trino.hdfs.TrinoFileSystemCache$FileSystemHolder.createFileSystemOnce(TrinoFileSystemCache.java:345)
	at io.trino.hdfs.TrinoFileSystemCache.getInternal(TrinoFileSystemCache.java:139)
	at io.trino.hdfs.TrinoFileSystemCache.get(TrinoFileSystemCache.java:92)
	at org.apache.hadoop.fs.ForwardingFileSystemCache.get(ForwardingFileSystemCache.java:38)
	at org.apache.hadoop.fs.FileSystem.get(FileSystem.java:479)
	at org.apache.hadoop.fs.Path.getFileSystem(Path.java:361)
	at org.apache.hudi.common.fs.FSUtils.getFs(FSUtils.java:109)
	... 43 more
```

## 解决办法
更改hudi连接配置，增加hdfs集群配置文件加载（core-site.xml,hdfs-site.xml）
```shell
connector.name=hudi
hive.metastore.uri=thrift://example.net:9083
hive.config.resources=/opt/hadoop/etc/hadoop/core-site.xml,/opt/hadoop/etc/hadoop/hdfs-site.xml
```

重启trino后再次通过DBeaver连接，可以正常读取hudi数据
