######基础组件部署参考

## 部署组件
* mysql
* redis
* neo4j
* nacos
* apisix
* minio
* elasticsearch
* kibana

## 环境准备
### 安装docker

```shell
#安装docker ，
#curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 无法用，去掉参数后可用
curl -fsSL https://get.docker.com | bash

#安装docker-compose
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.6.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

#配置docker私服和镜像代理
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"insecure-registries":["192.168.11.101:9999"],
"registry-mirrors": ["https://2ltiwjw8.mirror.aliyuncs.com"]
}

EOF
#刷新配置
sudo systemctl daemon-reload
sudo systemctl restart docker
```

安装结果验证，能打印就ok
```shell
[root@iZ2zefec0iuw1406rvb5i5Z docker]# docker version
Client: Docker Engine - Community
Version:           20.10.4
API version:       1.41
Go version:        go1.13.15
Git commit:        d3cb89e
Built:             Thu Feb 25 07:06:20 2021
OS/Arch:           linux/amd64
Context:           default
Experimental:      true

Server: Docker Engine - Community
Engine:
Version:          20.10.4
API version:      1.41 (minimum version 1.12)
Go version:       go1.13.15
Git commit:       363e9a8
Built:            Thu Feb 25 07:04:45 2021
OS/Arch:          linux/amd64
Experimental:     false
containerd:
Version:          1.4.3
GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
runc:
Version:          1.0.0-rc92
GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
docker-init:
Version:          0.19.0
GitCommit:        de40ad0
[root@iZ2zefec0iuw1406rvb5i5Z docker]# docker-compose version
docker-compose version 1.28.5, build c4eb3a1f
docker-py version: 4.4.4
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019

```

验证，下载镜像ok
```shell
[root@iZ2zefec0iuw1406rvb5i5Z docker]# docker login 192.168.11.101:9999
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
[root@iZ2zefec0iuw1406rvb5i5Z docker]# docker pull 192.168.11.101:9999/syyj/cmdb:v1.0.0-SNAPSHOT-master
v1: Pulling from syyj/cmdb
2d473b07cdd5: Pull complete
3ffd68c397bf: Pull complete
95fba8a11a2c: Pull complete
Digest: sha256:6d21228ff6d021b2412de8f376d60684d5332879c3acf23564882dd70618da1e
Status: Downloaded newer image for 192.168.11.101:9999/syyj/cmdb:v1.0.0-SNAPSHOT-master
192.168.11.101:9999/syyj/cmdb:v1.0.0-SNAPSHOT-master
[root@iZ2zefec0iuw1406rvb5i5Z ~]# docker images
REPOSITORY                                                TAG                            IMAGE ID       CREATED         SIZE
192.168.11.101:9999/syyj/cmdb:v1.0.0-SNAPSHOT-master      v1.0.0-SNAPSHOT-master         ab73bb1b895b   8 days ago      830MB

```

### 组件安装
#### mysql
**/opt/base/mysql目录下创建mysql.yml文件**
```shell
version: "3"
services:
mysql:
container_name: mysql
image: nacos/nacos-mysql:8.0.16
environment:
- MYSQL_ROOT_PASSWORD=root
- MYSQL_DATABASE=nacos_devtest
- MYSQL_USER=nacos
volumes:
- ./config/my.cnf:/etc/mysql/my.cnf:ro
- ./data:/var/lib/mysql:rw
ports:
- "3306:3306"
```
参数配置my.cnf
```shell
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Custom config should go here
!includedir /etc/mysql/conf.d/
```

**执行安装命令**
```shell
docker-compose up -d
```

#### redis
**/opt/base/redis目录下创建docker-compose.yml文件**
```shell
version: '3'
services:
redis:
restart: always
image: "redis:6.2"
container_name: redis
ports:
- "6379:6379"
volumes:
- ./data:/data
- ./config:/usr/local/etc/redis
```
配置参数redis.conf

执行安装命令
```shell
docker-compose up -d
```

#### neo4j
**/opt/base/neo4j目录下创建docker-compose.yml文件**
```shell
version: "3"
services:
neo4j:
image: neo4j:4.2.8
container_name: neo4j
environment:
- "NEO4J_AUTH=neo4j/tc123456"
ports:
- "7474:7474"
- "7687:7687"
volumes:
- ./data:/data
- ./logs:/logs
restart: always
```
执行安装命令
```shell
docker-compose up -d
```

#### nacos
**/opt/base/nacos目录下创建docker-compose.yml文件**
```shell
version: "3"
services:
nacos:
image: nacos/nacos-server:v2.0.4
container_name: nacos
restart: always
environment:
- JVM_XMS=512m
- JVM_XMX=512m
- PREFER_HOST_MODE=ip
- SPRING_DATASOURCE_PLATFORM=mysql
- MODE=standalone
- MYSQL_SERVICE_HOST=10.10.87.169
- MYSQL_SERVICE_PORT=3306
- MYSQL_SERVICE_USER=root
- MYSQL_SERVICE_PASSWORD=root
- MYSQL_SERVICE_DB_NAME=nacos_devtest
volumes:
- ./logs/:/home/nacos/logs
- ./plugins/:/home/nacos/plugins
network_mode: "host"
#    ports:
#      - 8848:8848
#      - 9848:9848
```
执行安装命令
```shell
docker-compose up -d
```

#### apisix
**/opt/base/apisxi目录下创建docker-compose.yml文件**
```shell
version: "3"

services:
apisix-dashboard:
image: apache/apisix-dashboard:2.9.0
restart: always
volumes:
- ./dashboard_conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml
ports:
- "9000:9000"
networks:
apisix:

apisix:
image: apache/apisix:2.10.0-alpine
restart: always
volumes:
- ./apisix_log:/usr/local/apisix/logs
- ./apisix_conf.yaml:/usr/local/apisix/conf/config.yaml:ro
depends_on:
- etcd
##network_mode: host
ports:
- "9080:9080/tcp"
- "9091:9091/tcp"
- "9443:9443/tcp"
- "9092:9092/tcp"
networks:
apisix:

etcd:
image: bitnami/etcd:3.4.15
restart: always
volumes:
- etcd_data:/bitnami/etcd
environment:
ETCD_ENABLE_V2: "true"
ALLOW_NONE_AUTHENTICATION: "yes"
ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379"
ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
ports:
- "2379:2379/tcp"
networks:
apisix:

networks:
apisix:
driver: bridge

volumes:
etcd_data:
driver: local
```
**apisix_conf.yaml**
```shell
apisix:
node_listen: 9080              # APISIX listening port
enable_ipv6: false

allow_admin:                  # http://nginx.org/en/docs/http/ngx_http_access_module.html#allow
- 0.0.0.0/0              # We need to restrict ip access rules for security. 0.0.0.0/0 is for test.

admin_key:
- name: "admin"
key: edd1c9f034335f136f87ad84b625c8f1
role: admin                 # admin: manage all configuration data
# viewer: only can view configuration data
- name: "viewer"
key: 4054f7cf07e344346cd3f287985e76a2
role: viewer

enable_control: true
control:
ip: "0.0.0.0"
port: 9092

etcd:
host:                           # it's possible to define multiple etcd hosts addresses of the same etcd cluster.
- "http://etcd:2379"     # multiple etcd address
prefix: "/apisix"               # apisix configurations prefix
timeout: 30                     # 30 seconds

plugin_attr:
prometheus:
export_addr:
ip: "0.0.0.0"
port: 9091

discovery:
nacos:
host:
- "http://nacos:nacos@10.10.87.168:8848"
prefix: "/nacos/v1/"
fetch_interval: 30    # default 30 sec
weight: 100           # default 100
timeout:
connect: 2000       # default 2000 ms
send: 2000          # default 2000 ms
read: 5000          # default 5000 ms
```
**dashboard_conf.yaml**
```shell
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

conf:
listen:
host: 0.0.0.0     # `manager api` listening ip or host name
port: 9000          # `manager api` listening port
allow_list:           # If we don't set any IP list, then any IP access is allowed by default.
- 0.0.0.0/0
etcd:
endpoints:          # supports defining multiple etcd host addresses for an etcd cluster
- "http://etcd:2379"
# yamllint disable rule:comments-indentation
# etcd basic auth info
# username: "root"    # ignore etcd username if not enable etcd auth
# password: "123456"  # ignore etcd password if not enable etcd auth
mtls:
key_file: ""          # Path of your self-signed client side key
cert_file: ""         # Path of your self-signed client side cert
ca_file: ""           # Path of your self-signed ca cert, the CA is used to sign callers' certificates
# prefix: /apisix     # apisix config's prefix in etcd, /apisix by default
log:
error_log:
level: warn       # supports levels, lower to higher: debug, info, warn, error, panic, fatal
file_path:
logs/error.log  # supports relative path, absolute path, standard output
# such as: logs/error.log, /tmp/logs/error.log, /dev/stdout, /dev/stderr
access_log:
file_path:
logs/access.log  # supports relative path, absolute path, standard output
# such as: logs/access.log, /tmp/logs/access.log, /dev/stdout, /dev/stderr
# log example: 2020-12-09T16:38:09.039+0800    INFO    filter/logging.go:46    /apisix/admin/routes/r1 {"status": 401, "host": "127.0.0.1:9000", "query": "asdfsafd=adf&a=a", "requestId": "3d50ecb8-758c-46d1-af5b-cd9d1c820156", "latency": 0, "remoteIP": "127.0.0.1", "method": "PUT", "errs": []}
authentication:
secret:
secret              # secret for jwt token generation.
# NOTE: Highly recommended to modify this value to protect `manager api`.
# if it's default value, when `manager api` start, it will generate a random string to replace it.
expire_time: 3600     # jwt token expire time, in second
users:                # yamllint enable rule:comments-indentation
- username: admin   # username and password for login `manager api`
password: admin
- username: user
password: user

plugins:                          # plugin list (sorted in alphabetical order)
- api-breaker
- authz-keycloak
- basic-auth
- batch-requests
- consumer-restriction
- cors
# - dubbo-proxy
- echo
# - error-log-logger
# - example-plugin
- fault-injection
- grpc-transcode
- hmac-auth
- http-logger
- ip-restriction
- jwt-auth
- kafka-logger
- key-auth
- limit-conn
- limit-count
- limit-req
# - log-rotate
# - node-status
- openid-connect
- prometheus
- proxy-cache
- proxy-mirror
- proxy-rewrite
- redirect
- referer-restriction
- request-id
- request-validation
- response-rewrite
- serverless-post-function
- serverless-pre-function
# - skywalking
- sls-logger
- syslog
- tcp-logger
- udp-logger
- uri-blocker
- wolf-rbac
- zipkin
- server-info
- traffic-split
```
  **执行安装命令**
```shell
docker-compose up -d
```

#### minio
**/opt/base/minio目录下创建docker-compose.yml文件**
```shell
version: '3.7'
services:
minio:
image: minio/minio:RELEASE.2022-01-04T07-41-07Z
command: server /data  --address ":9000" --console-address ":9090"
container_name: minio
ports:
- "9000:9000"
- "9090:9090"
environment:
MINIO_ROOT_USER: minio
MINIO_ROOT_PASSWORD: minio123
volumes:
- /data/minio/data:/data
network_mode: "host"
healthcheck:
test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
timeout: 20s
retries: 3
restart: always
```
执行安装命令
```shell
docker-compose up -d
```

#### elasticsearch
**/opt/base/elasticsearch目录下创建docker-compose.yml文件**
```shell
version: '3'
services:
elasticsearch:
restart: always
image: "192.168.11.101:9200/elasticsearch:7.12.1"
container_name: elasticsearch_log
ports:
- "9200:9200"
- "9300:9300"
environment:
- discovery.type=single-node
volumes:
- ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```
**配置参数elasticsearch.yml**
```shell
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```
执行安装命令
```shell
docker-compose up -d
```

#### kibana
**/opt/base/kibana目录下创建docker-compose.yml文件**
```shell
version: '3.7'
services:
kibana:
image: kibana:7.12.1
container_name: kibana
ports:
- "5601:5601"
environment:
- SERVER_NAME=kibana
- ELASTICSEARCH_HOSTS="http://10.10.87.169:9200"
- I18N_LOCALE="zh-CN"
volumes:
- /etc/localtime:/etc/localtime
network_mode: "host"
restart: always
deploy:
resources:
limits:
cpus: '2'
memory: 1G
```
**配置参数kibana.yml**
```shell
# Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://10.10.87.169:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
```
执行安装命令
```shell
docker-compose up -d
```
