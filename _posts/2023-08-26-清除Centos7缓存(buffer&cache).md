---
layout:     post
title:      脚本定时清除Centos7缓存(buffer/cache)
subtitle:   Centos7缓存定时清除
date:       2023-08-6
author:     zhmingyong
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Linux
    - 缓存清除
    - 定时清除
---

## 查看缓存
```
[root@bogon ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:            62G         26G         18G        2.0G         18G         34G
Swap:           15G         14M         15G
```

## 清除缓存脚本
```
#!/bin/bash
echo "开始清除缓存"
sync;sync;sync; # 将未写入的缓存写入硬盘，防止数据丢失
sleep 60 # 暂停 60秒
echo 1 > /proc/sys/vm/drop_caches
echo 2 > /proc/sys/vm/drop_caches
echo 3 > /proc/sys/vm/drop_caches
```

## 授权(增加可执行权限)
```
chmod +x /opt/cacheClean.sh
```

## 创建定时脚本
`crontab e` 打开定时器编辑
```
0 2/5 * * * /opt/cacheClean.sh
```