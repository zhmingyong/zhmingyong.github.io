---
layout:     post
title:      Trino HTTPS与密码认证实战
subtitle:   Trino密码认证实战
date:       2024-4-16
author:     zhmingyong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Trino
    - Https
    - 密码认证
---

## 概述
Trino支持TLS（传输层安全性）认证以确保在数据传输过程中的安全性。TLS认证是一种用于加密和保护数据传输的协议，它在客户端和服务器之间建立安全的通信通道，以防止中间人攻击和数据泄露。

以下是使用TLS认证的一些背景信息：

数据安全性：在数据分析和查询过程中，敏感数据的传输可能涉及到机密信息，如个人身份信息、财务数据等。TLS认证提供了一种加密通信的方式，确保数据在传输过程中不被未经授权的人或系统访问。

防止中间人攻击：TLS认证可以防止中间人攻击，其中攻击者试图在客户端和服务器之间的通信中插入恶意代码或窃取数据。通过建立安全通信通道并验证服务器的身份，TLS可以保护免受此类攻击的影响。

合规性要求：许多法规和合规性要求要求对敏感数据的传输进行加密。使用TLS认证有助于满足这些要求，并降低数据泄露的风险。

身份验证：TLS认证还允许客户端验证服务器的身份。服务器使用SSL证书来证明其身份，客户端可以验证此证书以确保连接到正确的服务器。这有助于防止恶意伪装服务器的攻击。

数据完整性：TLS还提供了数据完整性检查，以确保在传输过程中数据未被篡改或损坏。

要在Trino中启用TLS认证，你需要生成和配置SSL证书，然后在Trino的配置文件中指定证书的路径和其他相关设置。一旦配置完成，Trino服务器和客户端将能够建立安全的TLS连接，确保数据传输的机密性和完整性。这对于保护敏感数据以及满足安全性和合规性要求非常重要。

官方文档：https://trino.io/docs/current/security/tls.html

## 安装部署
略过

## 配置HTTPS
### 1）生成证书

如果已有证书，这一步就可以省略。

```bash
mkdir tls && cd tls
# 生成 CA 证书私钥
openssl genrsa -out trino.key 4096
# 生成 CA 证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=mytrino.com" \
 -key trino.key \
 -out trino.cert

# 合并
cat trino.key trino.cert > trino.pem
```

### 2）配置 Trino

在Trino的配置文件中，指定密钥库和密码，以便Trino服务器能够使用生成的证书和密钥。配置文件中的示例设置如下：

```bash
http-server.https.enabled=true
http-server.https.port=8443
http-server.https.keystore.path=/usr/local/trino/etc/tls/trino.pem
#节点之间不需要使用https通信，在内网部署无需，外网需打开，默认就是false
internal-communication.https.required=false
```

### 3）重启 Trino 服务
略

### 4）访问验证

![](https://zhmingyong.github.io/img/trino/trino-https-1.png)

## 密码认证

> 【温馨提示】要使用trino的权限控制，从官方文档上看，需要开启https，因此一定需要证书的。

### 1）开启密码认证

```bash
# etc/config.properties增加如下配置，开启密码认证
http-server.authentication.type=PASSWORD
# trino最新版本要求当启用trino的认证机制后，比如你开了kerberos，就必须要求trino节点内部通信提供一个shared-secret，这个secret你可以自定义。worker和coordinator的shared-secret保持一致即可。
#节点通信秘钥凭证
internal-communication.shared-secret=trino2024tc
```

### 2）创建密码认证配置文件

创建配置`etc/password-authenticator.properties`，此处可以配置使用LDAP或密码文件认证，这里测试采用密码文件认证。配置内容如下：  
`password-authenticator.properties`，里面需要用到`password.db`，这个文件就是在登录trino的时候使用到的用户名密码，需要`bcrypt`加密，可以去网络上找在线加密

```bash
password-authenticator.name=file
file.password-file=/usr/local/trino/etc/password.db
```

`password.db`，用户密码文件，一行就是一个用户名密码，用户密码用`:`隔开，比如我定义了一个`admin/123456`的用户。在使用`bcrypt`加密的时候，生成的密码可能是`$2a$`开头的，而根据trino文档内定义的`password`规范是需要`$2y$`开头的，这里需要注意。

这里提供一个在线bcrypt加密地址：[https://www.bejson.com/encrypt/bcrpyt_encode/](https://www.bejson.com/encrypt/bcrpyt_encode/)

`etc/password.db` 内容如下：

```bash
hadoop:$2y$10$7w7Km5YR9i1VONQD3X18mO/S54Jg.SmSxLiqX3KkwNkIiXWMfNlP.
```

### 3）重启 Trino 服务
略

### 4）访问验证

![](https://zhmingyong.github.io/img/trino/trino-password-1.png)

### 5）DBeaver 连接

![](https://zhmingyong.github.io/img/trino/trino-password-2.png)
