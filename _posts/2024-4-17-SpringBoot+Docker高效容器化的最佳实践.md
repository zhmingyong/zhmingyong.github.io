---
layout:     post
title:      SpringBoot+Docker高效容器化的最佳实践
subtitle:   高效容器化的最佳实践
date:       2024-4-17
author:     zhmingyong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - SpringBoot
    - Docker
---

# SpringBoot+Docker：高效容器化的最佳实践

首先为什么要使用Docker？

Docker是一个强大的工具，它允许开发者将他们的应用程序打包到容器中，以便可以在任何平台上轻松部署和运行。当涉及到对 Spring Boot 应用程序进行 Docker 化时，每个开发人员都应该遵循一些最佳实践，以确保应用程序平稳高效地运行。

**在本文中，我们将探讨这些最佳实践，并提供代码示例和说明，以帮助您对 Spring Boot 应用程序进行 Docker 化。**

作为一个 java 开发者，有很多用于支持 spring-boot 应用程序的基础官方镜像，我们需要关注镜像的大小，特别是当项目变大时。

## 使用正确的基础镜像

当对 Spring Boot 应用程序进行 Docker 化时，为您的应用程序选择正确的基础镜像非常重要。您可能知道 Docker 中的所有镜像都有 Linux 内核的基础层，因此我们不需要将这部分添加到我们的镜像中，因为我们的基础镜像提供了您的应用程序所需的底层内核和依赖项。选择正确的基础镜像有助于确保您的应用程序在 Docker 容器中平稳高效地运行。

对于 Spring Boot 应用程序，建议使用 OpenJDK 基础映像。OpenJDK 是 Java 开发工具包 (JDK) 的开源实现，提供 Java 运行时环境和 Java 开发工具。OpenJDK 基础映像有不同版本，例如 Java 8、Java 11 和 Java 16。以下是使用 OpenJDK 11 基础映像的 Dockerfile 示例：

```dockerfile
FROM openjdk:17-jdk-slim  
COPY target/springBootDockerized-0.0.1-SNAPSHOT.jar springBootDockerized-0.0.1-SNAPSHOT.jar  
ENTRYPOINT ["java" , "-jar" , "/springBootDockerized-0.0.1-SNAPSHOT.jar"]  
```

但非常重要的是，我们不需要 JDK，我们只需要 JRE java 运行时环境 我建议在OpenJDK官方链接中使用 JRE 层，您可以找到以下内容：

```bash
eclipse-temurin  
```

作为示例 spring-boot 应用程序，添加一个 Dockerfile 到 root，如下所示：

```dockerfile
#dockerized 使用 JDK 的不好做法  
#FROM openjdk:17-jdk-slim  
#COPY target/springBootDockerized-0.0.1-SNAPSHOT.jar springBootDockerized-0.0.1-SNAPSHOT.jar  
#ENTRYPOINT ["java" , "-jar" , "/springBootDockerized-0.0.1-SNAPSHOT.jar"]  
  
  
FROM eclipse-temurin:17.0.5_8-jre-focal as builder  
WORKDIR extracted  
ADD ./target/*.jar app.jar  
RUN java -Djarmode=layertools -jar app.jar extract  
  
FROM eclipse-temurin:17.0.5_8-jre-focal  
WORKDIR application  
COPY --from=builder extracted/dependencies/ ./  
COPY --from=builder extracted/spring-boot-looder/ ./  
COPY --from=builder extracted/snapshot-dependencies/ ./  
COPY --from=builder extracted/application/ ./  
  
EXPOSE 8085  
  
#在 springboot 3.2 之前使用它  
#ENTRYPOINT ["java" , "org.springframework.boot.loader.JarLauncher"]  
#在 springboot 3.2 之后使用它  
  
ENTRYPOINT ["java" , "org.springframework.boot.loader.launch.JarLauncher"]  
```

在此示例中，第一阶段使用 Maven 基础映像来构建 Spring Boot 应用程序并生成 jar 文件。第二阶段使用 OpenJDK slim 基础镜像，它是基础镜像的较小版本，仅包含 Java 运行时环境。

该`COPY --from=build`指令将jar文件从第一阶段复制到第二阶段，该`ENTRYPOINT`指令指定容器启动时应该运行的命令。

第一部分指令的含义：

- **java：** 这是运行Java应用程序或执行Java字节码的命令。
    
- **\-Djarmode=layertools：** 这是一个系统属性，它使用-D标志指定。它将HRIMARMODE属性的值设置为更高级的LayerTools。这是启用“layertools”模式来操作模块化 JAR 文件中的“层”的另一种方法。
    
- **\-jar app.jar：** 指定要执行的 JAR 文件名为“app.jar”。该`-jar`选项指示指定的文件是可执行的 JAR 文件。
    
- **extract：** 这是在 JAR 文件中传递给应用程序的参数或命令。它指示应用程序执行特定操作，在本例中是提取 JAR 文件的内容。
    

通过这种方式使用多阶段构建，我们可以创建一个精简的 Docker 映像，其中仅包含运行 Spring Boot 应用程序所需的依赖项和文件。通过这样做，我们可以减小图像的大小并提高应用程序的性能。

另一种方法是使用 Build-pack.io，它会在您的 pom 中自动为您生成图像，并将其添加到插件标签中：

```xml
<build>  
    <plugins>  
        <plugin>  
            <configuration>  
                <image>  
                    <name>kia/test</name>  
                </image>  
            </configuration>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
        </plugin>  
    </plugins>  
</build>
``` 

之后运行此命令：

```bash
mvn spring-boot:build-image  
```

使用这个命令 spring boot 可以完美地为你制作镜像。

## 使用环境变量

当对 Spring Boot 应用程序进行 Docker 化时，使用环境变量来配置应用程序非常重要。使用环境变量允许您更改应用程序的配置，而无需重建 Docker 映像。

Spring Boot 应用程序可以使用`application.properties`或`application.yml`文件来指定配置属性。这些属性可以在运行时使用环境变量覆盖，Spring Boot 会自动将其映射到属性。下面是一个示例 Dockerfile，它设置一个环境变量来配置 Spring Boot 应用程序的活动配置文件：

```dockerfile
FROM openjdk:11  
ENV SPRING_PROFILES_ACTIVE=production  
COPY target/my-application.jar app.jar  
ENTRYPOINT ["java", "-jar", "/app.jar"]  
```

在本例中，我们将`SPRING_PROFILES_ACTIVE`环境变量设置为生产环境变量，这将激活Spring Boot应用程序中的生产配置文件。

当容器启动时，`ENTRYPOINT`指令中指定的java命令将与`-jar`选项一起运行，以启动Spring Boot应用程序。由于我们设置了`SPRING_PROFILES_ACTIVE`环境变量，应用程序将自动使用生产环境。

## 使用健康检查

对 Spring Boot 应用程序进行 Docker 化时，使用运行状况检查来监控应用程序的运行状况并确保其正确运行非常重要。健康检查可用于检测应用程序何时不健康，并根据应用程序的健康状况自动执行恢复或扩展。

要在Docker映像中添加健康检查，您可以使用Dockerfile中的`HEALTHCHECK`指令。`HEALTHCHECK`指令告诉Docker如何检查应用程序的运行状况。下面是一个示例Dockerfile，它为Spring Boot应用程序添加了健康检查：

```dockerfile
FROM openjdk:11  
COPY target/my-application.jar app.jar  
HEALTHCHECK --interval=5s \  
            --timeout=3s \  
            CMD curl -f http://localhost:8080/actuator/health || exit 1  
ENTRYPOINT ["java", "-jar", "/app.jar"]  
```

在此示例中，我们使用`HEALTHCHECK`指令来检查 Spring Boot 应用程序的运行状况。该`--interval`选项指定运行状况检查的频率，以及`--timeout`指定等待响应的时间。该`CMD`指令运行健康检查命令，这是curl检查`/actuator/health`应用程序端点的命令。

运行容器时，可以使用`docker ps`命令查看容器的健康状态：

```bash
$ docker ps  
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS          PORTS                    NAMES  
e8e1a6440e5e   my-application:1.0   "java -jar /app.jar"     5 seconds ago   Up 4 seconds    0.0.0.0:8080->8080/tcp   my-application  
$ docker inspect --format='{{json .State.Health}}' my-application  
{"Status":"healthy","FailingStreak":0,"Log":[{"Start":"2023-03-25T09:21:08.272130387Z","End":"2023-03-25T09:21:08.310105965Z","ExitCode":0,"Output":"\n"}]}  
```

在此示例中，该`docker ps`命令显示容器已启动并在端口8080上运行。该`docker inspect`命令显示容器的健康状态，当前为healthy。如果健康检查失败，容器将被标记为unhealthy，您可以使用 Docker Compose 或 Kubernetes 等工具自动恢复或扩展容器。

## 使用 Docker 缓存

当对 Spring Boot 应用程序进行 Docker 化时，使用 Docker 缓存来加快构建过程并减少构建新 Docker 映像所需的时间非常重要。Docker 缓存允许您重用之前构建的 Docker 镜像层，从而避免每次构建新镜像时都需要重建这些层。下面是一个使用 Docker 缓存来加速构建过程的 Dockerfile 示例：

```dockerfile
FROM openjdk:11 as builder  
WORKDIR /app  
COPY pom.xml .  
RUN mvn dependency:go-offline  
  
COPY src/ ./src/  
RUN mvn package -DskipTests  
  
FROM openjdk:11  
COPY --from=builder /app/target/my-application.jar app.jar  
ENTRYPOINT ["java", "-jar", "/app.jar"]  
```

每一步都假设一个缓存在Docker注册表层的阶段，

在此示例中，我们使用多阶段构建，首先在单独的层中构建 Spring Boot 应用程序，然后将构建的 jar 文件复制到最终镜像中。通过在构建过程中使用单独的层，我们可以利用 Docker 缓存来避免每次构建新镜像时重建依赖项。

- 构建过程的第一阶段使用`openjdk:11`基础镜像并复制pom.xml文件到容器。然后它运行`mvn dependency:go-offline`命令下载应用程序所需的所有依赖项。该命令确保所有必需的依赖项在本地可用，这将加快后续构建的速度。
    
- 构建过程的第二阶段使用`openjdk:11`基础映像并将源代码复制到容器中。然后它运行`mvn package`命令来构建应用程序 jar 文件。由于我们在上一阶段已经下载了依赖项，因此 Docker 将使用缓存层并跳过依赖项下载步骤。
    
- 最后，该`COPY --from=builder`指令将构建的 jar 文件从构建器阶段复制到最终映像，并且该`ENTRYPOINT`指令指定容器启动时应运行的命令。
    

## 使用 .dockerignore 文件

当对 Spring Boot 应用程序进行 Docker 化时，使用`.dockerignore`文件从 Docker 构建上下文中排除不必要的文件和目录非常重要。通过使用`.dockerignore`文件，您可以排除 Docker 镜像不需要的文件和目录，从而减少构建上下文的大小并提高构建性能。以下是Spring Boot 应用程序的 `.dockerignore` 示例文件：

```bash
# 忽略根目录下的所有文件  
*  
# 包含 src 目录  
!src/  
# 包含 pom.xml 文件  
!pom.xml  
# 排除目标目录及其内容  
target/  
```

在此示例中，我们使用该`.dockerignore`文件排除根目录 (`*`) 中的所有文件，除了构建 Spring Boot 应用程序所需的`src/`目录和`pom.xml`文件。我们还排除了`target/`目录，该目录包含构建的工件，Docker映像不需要该目录。

通过使用`.dockerignore`文件，我们可以减少构建上下文的大小并提高构建性能。Docker只会复制`.dockerignore`构建上下文中包含的文件和目录，并且会忽略文件中排除的文件和目录。

> “
> 
> 使用`.dockerignore`文件是对 Spring Boot 应用程序进行 Docker 化的良好实践，因为它有助于确保尽可能高效、快速地构建 Docker 映像。

此外，使用`.dockerignore`文件还可以帮助提高Docker 镜像的安全性。通过排除不必要的文件和目录，您可以减少 Docker 映像的攻击面，并最大限度地降低暴露敏感信息或凭据的风险。例如，如果您在构建目录中存储了配置文件或凭据，则将它们排除在`.dockerignore`文件中将阻止它们包含在 Docker 映像中。

> “
> 
> 还值得注意的是，该`.dockerignore`文件遵循与`.gitignore`文件类似的语法，用于从 Git 存储库中排除文件和目录。如果您熟悉该`.gitignore`文件，`.dockerignore`文件的使用是零学习成本。

总之，使用`.dockerignore`文件是 Docker 化 Spring Boot 应用程序的良好实践。它可以帮助减少构建上下文的大小、提高构建性能并提高 Docker 映像的安全性。

## 使用标签

对 Spring Boot 应用程序进行 Docker 化时，使用标签将元数据添加到 Docker 映像非常重要。标签是键值对，可以添加到 Docker 映像中以提供有关映像的附加信息，例如版本、维护者或构建日期。下面是一个示例 Dockerfile，它使用标签将元数据添加到 Spring Boot 应用程序：

```dockerfile
FROM openjdk:11  
LABEL maintainer="John Doe <john.doe@example.com>"  
LABEL version="1.0"  
LABEL description="My Spring Boot application"  
COPY target/my-application.jar app.jar  
ENTRYPOINT ["java", "-jar", "/app.jar"]  
```

在此示例中，我们使用`LABEL`指令将元数据添加到 Docker 映像。我们为镜像的维护者、版本和描述添加了标签。这些标签提供有关 Docker 映像的附加信息，并帮助用户了解映像包含的内容及其构建方式。

您可以使用`docker inspec`t命令查看 Docker 镜像的标签：

```bash
$ docker inspect my-application  
[  
    {  
        "Id": "sha256:...",  
        "RepoTags": [  
            "my-application:latest"  
        ],  
        "Labels": {  
            "maintainer": "John Doe <john.doe@example.com>",  
            "version": "1.0",  
            "description": "My Spring Boot application"  
        },  
        ...  
    }  
]  
```

在本例中，`docker inspect`命令显示`my-application` Docker镜像的标签。标签提供有关镜像的其他信息，可以帮助用户了解镜像是如何构建的以及如何使用它。

以这种方式使用标签可以帮助提高 Docker 镜像的可用性和可维护性。通过向 Docker 映像添加元数据，您可以帮助用户了解镜像包含的内容及其构建方式。随着时间的推移，此信息对于调试、故障排除和维护 Docker 镜像非常有用。