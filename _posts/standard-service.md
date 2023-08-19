######标准服务



## 服务部署

#### 变量

- 镜像所属项目名称，例：`projectName=cup`
- 部署服务名称，例:`serviceName=cup-desk`
- 服务镜像版本，例：`version=v3.2.0.RELEASE-master`
- 服务端口，例：`servicePort=8105`
- 服务启动类，例：`className=cup.core.desk.DeskApplication`

变量请根据实际形况填写到docker-compose文件中，此处只是为说明方便而提出来定义



`environment`环境变量值请根据镜像实际情况填写

```yaml
version: '3'
services:
  ${serviceName}:
    image: "192.168.11.101:9999/${projectName}/${serviceName}:${version}"
    container_name: ${serviceName}
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
      - ${servicePort}:${servicePort}
    network_mode: "host"
    entrypoint:
      - java
      - -cp
      - ${serviceName}/.:${serviceName}/lib/*:${serviceName}/${serviceName}.jar
      - ${className}
```

