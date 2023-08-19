######基础服务



## 当前基础服务部署



以下服务可根据实际情况按需删减，只保留需要的服务进行部署即可，**`其中的项目名称及版本号请确认好`**

```yaml
version: '3'
services:
  cup-auth:
    image: "192.168.11.101:9999/cup/cup-auth:v3.2.0.RELEASE-master"
    container_name: cup-auth
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.client.addr=192.168.3
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8100:8100
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-auth/.:cup-auth/lib/*:cup-auth/cup-auth.jar
      - cup.core.auth.AuthApplication


  cup-desk:
    image: "192.168.11.101:9999/cup/cup-desk:v3.2.0.RELEASE-master"
    container_name: cup-desk
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8105:8105
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-desk/.:cup-desk/lib/*:cup-desk/cup-desk.jar
      - cup.core.desk.DeskApplication


  cup-user:
    image: "192.168.11.101:9999/cup/cup-user:v3.2.0.RELEASE-master"
    container_name: cup-user
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8102:8102
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-user/.:cup-user/lib/*:cup-user/cup-user.jar
      - cup.core.system.user.UserApplication


  cup-system:
    image: "192.168.11.101:9999/cup/cup-system:v3.2.0.RELEASE-master"
    container_name: cup-system
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8106:8106
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-system/.:cup-system/lib/*:cup-system/cup-system.jar
      - cup.core.system.SystemApplication


  cup-log:
    image: "192.168.11.101:9999/cup/cup-log:v3.2.0.RELEASE-master"
    container_name: cup-log
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8103:8103
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-log/.:cup-log/lib/*:cup-log/cup-log.jar
      - cup.core.log.LogApplication


  cup-report:
    image: "192.168.11.101:9999/cup/cup-report:v3.2.0.RELEASE-master"
    container_name: cup-report
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8108:8108
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-report/.:cup-report/lib/*:cup-report/cup-report.jar
      - cup.core.report.ReportApplication


  cup-resource:
    image: "192.168.11.101:9999/cup/cup-resource:v3.2.0.RELEASE-master"
    container_name: cup-resource
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8010:8010
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-resource/.:cup-resource/lib/*:cup-resource/cup-resource.jar
      - cup.core.resource.ResourceApplication


  cup-swagger:
    image: "192.168.11.101:9999/cup/cup-swagger:v3.2.0.RELEASE-master"
    container_name: cup-swagger
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 18000:18000
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-swagger/.:cup-swagger/lib/*:cup-swagger/cup-swagger.jar
      - cup.core.swagger.SwaggerApplication


  cup-flow:
    image: "192.168.11.101:9999/cup/cup-flow:v3.2.0.RELEASE-master"
    container_name: cup-flow
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
      - cup.redis.host=10.10.87.169
      - cup.redis.port=6379
    privileged: true
    restart: always
    ports:
      - 8008:8008
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-flow/.:cup-flow/lib/*:cup-flow/cup-flow.jar
      - cup.core.flow.FlowApplication


  cup-admin:
    image: "192.168.11.101:9999/cup/cup-admin:v3.2.0.RELEASE-master"
    container_name: cup-admin
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
    privileged: true
    restart: always
    ports:
      - 7002:7002
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-admin/.:cup-admin/lib/*:cup-admin/cup-admin.jar
      - cup.core.admin.AdminApplication


  cup-xxljob:
    image: "192.168.11.101:9999/cup/cup-xxljob:v3.2.0.RELEASE-master"
    container_name: cup-xxljob
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
    privileged: true
    restart: always
    ports:
      - 7008:7008
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-xxljob/.:cup-xxljob/lib/*:cup-xxljob/cup-xxljob.jar
      - cup.core.job.executor.JobApplication


  cup-xxljob-admin:
    image: "192.168.11.101:9999/cup/cup-xxljob-admin:v3.2.0.RELEASE-master"
    container_name: cup-xxljob-admin
    environment:
      - TZ=Asia/Shanghai
      - cup.nacos.addr=10.10.87.168:8848
      - cup.client.addr=192.168.3
      - cup.nacos.namespace=public
      - cup.nacos.group=DEFAULT_GROUP
      - cup.datasource.url=jdbc:mysql://10.10.87.169:3306/cup?useSSL=false&useUnicode=true&characterEncoding=utf-8
      - cup.datasource.username=root
      - cup.datasource.password=root
    privileged: true
    restart: always
    ports:
      - 7009:7009
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - cup-xxljob-admin/.:cup-xxljob-admin/lib/*:cup-xxljob-admin/cup-xxljob-admin.jar
      - com.xxl.job.admin.JobAdminApplication

```

