---
layout:     post
title:      CentOS、Ubuntu、Debian、Alpine更换国内源方法
subtitle:   更换国内源
date:       2023-12-08
author:     zhmingyong
header-img: img/post-bg-debug.png
catalog: true
tags:
    - CentOS
    - Ubuntu
    - Debain
    - Alpine
    - 更换国内源
---

一般来说，Linux系统、Docker、NPM的国内源可以大幅加速软件和镜像包的下载速度，同时也可以避免一些网络问题。

国内的源通常是由国内的各大云服务商提供的，这些云服务商在本地都有自己的加速服务器和缓存系统，可以更快地下载镜像。另外，由于地理位置的原因，国内的源通常比国外的源更稳定，也更容易受到国内用户的访问。因此，更换国内源可以提高开发效率和稳定性，减少不必要的网络问题。


## CentOS换源

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
或者
```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

## Debian换源

### debian-9归档源

```shell
echo "deb http://archive.debian.org/debian/ stretch main contrib non-free" > /etc/apt/sources.list \
 && echo "deb http://archive.debian.org/debian/ stretch-proposed-updates main contrib non-free" >> /etc/apt/sources.list \
 && echo "deb http://archive.debian.org/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list
```
### 阿里云镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list
```
### 腾讯云镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.tencent.com@g" /etc/apt/sources.list
```
### 网易镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.163.com@g" /etc/apt/sources.list
```
### 华为镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
```
### 清华大学镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
```
### 中科大镜像站

```shell
sed -i "s@http://\(deb\|security\).debian.org@http://mirrors.ustc.edu.cn@g" /etc/apt/sources.list
```

## Ubuntu换源

### 阿里云镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.aliyun.com@g" /etc/apt/sources.list
```
### 腾讯云镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.tencent.com@g" /etc/apt/sources.list
```
### 网易镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.163.com@g" /etc/apt/sources.list
```
### 华为镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.huaweicloud.com@g" /etc/apt/sources.list
```
### 清华大学镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
```
### 中科大镜像站

```shell
sed -i "s@\(archive\|security\).ubuntu.com@mirrors.ustc.edu.cn@g" /etc/apt/sources.list
```

## Alpine linux换源

### 阿里云镜像站

```shell
sed -i "s@dl-cdn.alpinelinux.org@mirrors.aliyun.com@g" /etc/apk/repositories
```
### 华为镜像站

```shell
sed -i "s@dl-cdn.alpinelinux.org/@repo.huaweicloud.com/@g" /etc/apk/repositories
```

## Pip换源

### 阿里云

```shell
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
```
### 中国科技大学

```shell
pip config set global.index-url https://pypi.mirrors.ustc.edu.cn/simple/
```
### 豆瓣(douban)

```shell
pip config set global.index-url https://pypi.douban.com/simple/
```
### 清华大学

```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/
```
### 中国科学技术大学

```shell
pip config set global.index-url https://pypi.mirrors.ustc.edu.cn/simple/
```
### 华为

```shell
pip config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
```
## Conda添加源

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ \
 && conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ \
 && conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/ \
 && conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/
```

## Npm换源

### 淘宝

```shell
npm config set registry http://registry.npmmirror.com
```
### 华为

```shell
npm config set registry https://repo.huaweicloud.com/repository/npm/
```
### cnpmjs

```shell
npm config set registry http://r.cnpmjs.org
```
## GO换源

### 阿里云

```shell
go env -w GO111MODULE=on GOPROXY=https://mirrors.aliyun.com/goproxy/,direct GONOSUMDB=*
```
### 七牛

```shell
go env -w GO111MODULE=on GOPROXY=https://goproxy.cn,direct GONOSUMDB=*
```
### 华为

```shell
go env -w GO111MODULE=on GOPROXY=https://repo.huaweicloud.com/repository/goproxy/ GONOSUMDB=*
```
## Dockerfile非交互，修改时区

```shell
ENV TIME_ZONE Asia/Shanghai
RUN apt-get update \
&& DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata \
    && ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata
```
## Github访问加速，URL前缀添加

```shell
https://ghproxy.com/
```