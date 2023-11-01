---
layout:     post
title:      OpenFeign支持PATCH请求
subtitle:   OpenFeign支持PATCH请求
date:       2023-11-1
author:     zhmingyong
header-img: img/post-bg-debug.png
catalog: true
tags:
    - OpenFeign
    - Patch
---

# OpenFeign支持PATCH请求

## OpenFeign调用第三方接口
```java
@FeignClient(name = "FlinkClusterManagerFeign", contextId = "FlinkJobsManagerFeign", url = "http://localhost:8081")
public interface FlinkJobsManagerFeign {

    @GetMapping(value = "/jobs/overview", consumes = "application/json;charset=utf-8", produces = "application/json;charset=utf-8")
    FlinkJobs jobs();
}
```

## OpenFeign发起PATCH请求
```java
@FeignClient(name = "FlinkClusterManagerFeign", contextId = "FlinkJobsManagerFeign", url = "http://localhost:8081")
public interface FlinkJobsManagerFeign {

    @PatchMapping(value = "/jobs/{jobId}?mode=cancel")
    void jobCancel(@PathVariable(value = "jobId") String jobId);
}
```

## 调用异常
```shell
2023-11-01 09:48:37,027[ERROR]org.springframework.scheduling.support.TaskUtils$LoggingErrorHandler:Unexpected error occurred in scheduled task
feign.RetryableException: Invalid HTTP method: PATCH executing PATCH http://localhost:8081/jobs/dbec0207e33d5850ded606c283774a50?mode=cancel
	at feign.FeignException.errorExecuting(FeignException.java:268)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:131)
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:91)
	at feign.ReflectiveFeign$FeignInvocationHandler.invoke(ReflectiveFeign.java:100)
	at com.sun.proxy.$Proxy166.jobCancel(Unknown Source)
	at noc.dap.mission.service.component.MissionProcessDebugComponent.lambda$timedExitDebugging$3(MissionProcessDebugComponent.java:583)
	at java.util.concurrent.ConcurrentHashMap$KeySetView.forEach(ConcurrentHashMap.java:4647)
	at noc.dap.mission.service.component.MissionProcessDebugComponent.timedExitDebugging(MissionProcessDebugComponent.java:556)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84)
	at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset$$$capture(FutureTask.java:308)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
Caused by: java.net.ProtocolException: Invalid HTTP method: PATCH
	at java.net.HttpURLConnection.setRequestMethod(HttpURLConnection.java:440)
	at sun.net.www.protocol.http.HttpURLConnection.setRequestMethod(HttpURLConnection.java:558)
	at feign.Client$Default.convertAndSend(Client.java:171)
	at feign.Client$Default.execute(Client.java:105)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:121)
	... 20 common frames omitted
```

## 解决办法
 引入`feign-httpclient`依赖以支持PATCH
 ```xml
 <dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-httpclient</artifactId>
	<version>13.0</version>
</dependency>
 ```