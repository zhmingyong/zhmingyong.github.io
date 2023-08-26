---
layout:     post
title:      大数据集群Kerberos权限认证
subtitle:   Kerberos权限认证p配置验证
date:       2023-08-6
author:     zhmingyong
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Linux
    - 缓存清除
    - 定时清除
---

# 大数据集群Kerberos权限认证

------

## Kerberos简介

### 概述
Kerberos是一种计算机网络授权协议，用来在非安全网络中，对个人通信以安全的手段进行身份验证。

软件设计上采用客户端/服务器结构，并且能够相互认证，即客户端和服务器端均可对对方身份进行认证。
可以用于防窃听、防止重放攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。

### 概念

#### KDC
密钥分发中心，负责管理发放票据，记录授权

#### 域
Kerberos管理领域的标识。

#### principal
当每添加一个用户或服务的时候需要向kdc添加一条principal，principal的形式为:`主名称/实例名@领域名`。

#### 主名称
可以是用户名或服务名，可以是单词host,表示是用于提供各种网络服务(如hdfs,yarn,hive)的主体

#### 实例名
可以理解为主机名

#### 领域
Kerberos的域

### Kerberos认证原理

#### 客户端初始验证
TGT只是KDC认同客户端的票证

![](img/kerberos_1.png)

#### 获取服务访问
![](img/kerberos_2.png)

------


## Kerberos安装部署

Kerberos主节点（Kadmin，KDC）执行如下命令

```shell
yum install -y krb5-server krb5-libs krb5-workstation
```

Kerberos从节点（只使用Kerberos认证）执行如下命令

```shell
yum install -y krb5-devel krb5-workstation
```

所有节点配置krb5.conf(/etc/krb5.conf)

```shell
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
# renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = HADOOP.COM
# default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 HADOOP.COM = {
  kdc = hadoop1
  admin_server = hadoop1
 }

[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```

注意：此处需要注释配置项`renew_lifetime`,不然会引起主体认证找不到用户密码的异常;注释`default_ccache_name`配置项，避免引起spark、dolphinscheduler等读写hdfs异常

所有节点配置kdc.conf(/var/kerberos/krb5kdc/kdc.conf)

```shell
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 HADOOP.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```



------

## HADOOP配置kerberos认证

### keytab准备

hadoop部署安装时已创建hadoop集群相关用户，此处略过

创建认证主体

执行命令`addprinc nn/hadoop1`创建主体

```shell
[root@hadoop1 hadoop]# kadmin.local
Authenticating as principal root/admin@HADOOP.COM with password.
kadmin.local:  listprincs
K/M@HADOOP.COM
admin/admin@HADOOP.COM
dn/hadoop1@HADOOP.COM
hadoop/hadoop1@HADOOP.COM
jhs/hadoop@HADOOP.COM
jn/hadoop1@HADOOP.COM
kadmin/admin@HADOOP.COM
kadmin/changepw@HADOOP.COM
kadmin/hadoop1@HADOOP.COM
kiprop/hadoop1@HADOOP.COM
krbtgt/HADOOP.COM@HADOOP.COM
nm/hadoop@HADOOP.COM
nn/hadoop1@HADOOP.COM
rm/hadoop@HADOOP.COM
snn/hadoop1@HADOOP.COM
web/hadoop@HADOOP.COM
```

生成keytab

```shell
kadmin.local:  xst -k /home/hadoop/all.keytab dn/hadoop1@HADOOP.COM hadoop/hadoop1@HADOOP.COM jhs/hadoop@HADOOP.COM jn/hadoop1@HADOOP.COM nm/hadoop@HADOOP.COM nn/hadoop1@HADOOP.COM rm/hadoop@HADOOP.COM snn/hadoop1@HADOOP.COM web/hadoop@HADOOP.COM
```

keytab授权

```shell
chown hadoop:hadoop /home/hadoop/all.keytab
chmod 660 /home/hadoop/all.keytab
```

### 证书准备

生成证书

密码按实际输入，例：`123456`

```shell
[root@hadoop1 hadoop]# keytool -keystore /home/hadoop/keystore -alias jetty -genkey -keyalg RSA
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  T
What is the name of your organizational unit?
  [Unknown]:  T
What is the name of your organization?
  [Unknown]:  T
What is the name of your City or Locality?
  [Unknown]:  T
What is the name of your State or Province?
  [Unknown]:  T
What is the two-letter country code for this unit?
  [Unknown]:  T
Is CN=T, OU=T, O=T, L=T, ST=T, C=T correct?
  [no]:  yes

Enter key password for <jetty>
        (RETURN if same as keystore password):  
Re-enter new password: 

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /home/hadoop/keystore -destkeystore /home/hadoop/keystore -deststoretype pkcs12".
```

授权

```shell
chown root:hadoop /home/hadoop/keystore
chmod 660 /home/hadoop/keystore
```

`注意:keytab和证书（keystore）生成后需要同步到hadoop集群所有节点`

### HDFS认证配置

#### core-site配置添加如下内容

```xml
    <!-- 以下参数用于配置Kerberos -->
    <property>
        <name>hadoop.security.authentication</name>
        <value>kerberos</value>
        <description>此参数设置集群的认证类型,默认值是"simple"。当使用Kerberos进行身份验证时，请设置为"kerberos".</description>
    </property>

    <property>
        <name>hadoop.security.authorization</name>
        <value>true</value>
        <description>此参数用于确认是否启用安全认证,默认值为"false",我们需要启用该功能方能进行安全认证.</description>
    </property>

    <property>
        <name>hadoop.security.auth_to_local</name>
        <value>
             RULE:[2:$1@$0](nn/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](dn/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](jn/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](snn/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](rm/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](nm/.*@.*HADOOP.COM)s/.*/hadoop/
             RULE:[2:$1@$0](jhs/.*@.*HADOOP.COM)s/.*/hadoop/
             DEFAULT
        </value>
        <description>此参数指定如何使用映射规则将Kerberos主体名映射到OS用户名.</description>
    </property>

    <property>
        <name>hadoop.rpc.protection</name>
        <value>authentication</value>
        <description>此参数指定保护级别，有三种可能,分别为authentication(默认值，表示仅客户端/服务器相互认值),integrity(表示保证数据的完整性并进行身份验证),privacy(进行身份验证并保护数据完整性，并且还加密在客户端与服务器之间传输的数据)</description>    
   </property>
```

#### hdfs-site配置添加如下内容

```xml
    <!-- 使用以下配置参数配置Kerberos服务主体 -->
    <property>
        <name>dfs.namenode.kerberos.principal</name>
        <value>nn/_HOST@HADOOP.COM</value>
        <description>此参数指定NameNode的Kerberos服务主体名称。通常将其设置为nn/_HOST@REALM.TLD。每个NameNode在启动时都将_HOST替换为其自己的标准主机名。_HOST占位符允许在HA设置中的两个NameNode上使用相同的配置设置。</description>    
　　</property>

    <property>
        <name>dfs.secondary.namenode.kerberos.principal</name>
        <value>snn/_HOST@HADOOP.COM</value>
        <description>此参数指定Secondary NameNode的Kerberos主体名称。</description>
    </property>

    <property>
        <name>dfs.web.authentication.kerberos.principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
        <description>NameNode用于WebHDFS SPNEGO身份验证的服务器主体。启用WebHDFS和安全性时需要。</description>
    </property>
   
    <property>
        <name>dfs.namenode.kerberos.internal.spnego.principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
        <description>启用Kerberos安全性时，NameNode用于Web UI SPNEGO身份验证的服务器主体。若不设置该参数，默认值为"${dfs.web.authentication.kerberos.principal}"</description>
    </property>
   
    <property>
        <name>dfs.secondary.namenode.kerberos.internal.spnego.principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
        <description>启用Kerberos安全性时，Secondary NameNode用于Web UI SPNEGO身份验证的服务器主体。与其他所有Secondary NameNode设置一样，在HA设置中将忽略它。默认值为"${dfs.web.authentication.kerberos.principal}"</description>    
　　</property>

    <property>
        <name>dfs.datanode.kerberos.principal</name>
        <value>dn/_HOST@HADOOP.COM</value>
        <description>此参数指定DataNode服务主体。通常将其设置为dn/_HOST@REALM.TLD。每个DataNode在启动时都将_HOST替换为其自己的标准主机名。_HOST占位符允许在所有DataNode上使用相同的配置设置</description>    
　　</property>

    <property>
        <name>dfs.block.access.token.enable</name>
        <value>true</value>
        <description>如果为"true"，则访问令牌用作访问数据节点的功能。如果为"false"，则在访问数据节点时不检查访问令牌。默认值为"false"</description>
    </property>

    <!-- 使用以下配置参数指定keytab文件 -->
    <property>
        <name>dfs.web.authentication.kerberos.keytab</name>
        <value>/opt/keytabs/keytab</value>
        <description>http服务主体的keytab文件位置,即"dfs.web.authentication.kerberos.principal"对应的主体的密钥表文件。</description>
    </property>

    <property>
        <name>dfs.namenode.keytab.file</name>
        <value>/opt/keytabs/keytab</value>
        <description>每个NameNode守护程序使用的keytab文件作为其服务主体登录。主体名称使用"dfs.namenode.kerberos.principal"配置。</description>
    </property>

    <property>
        <name>dfs.datanode.keytab.file</name>
        <value>/opt/keytabs/keytab</value>
        <description>每个DataNode守护程序使用的keytab文件作为其服务主体登录。主体名称使用"dfs.datanode.kerberos.principal"配置。</description>
    </property>

    <property>
        <name>dfs.secondary.namenode.keytab.file</name>
        <value>/opt/keytabs/keytab</value>
        <description>每个Secondary Namenode守护程序使用的keytab文件作为其服务主体登录。主体名称使用"dfs.secondary.namenode.kerberos.principal"配置。</description>
    </property>

    <!-- DataNode SASL配置,若不指定可能导致DataNode启动失败 -->
    <property>
        <name>dfs.data.transfer.protection</name>
        <value>integrity</value>
        <description>逗号分隔的SASL保护值列表，用于在读取或写入块数据时与DataNode进行安全连接。可能的值为:"authentication"(仅表示身份验证，没有完整性或隐私), "integrity"(意味着启用了身份验证和完整性)和"privacy"(意味着所有身份验证，完整性和隐私都已启用)。如果dfs.encrypt.data.transfer设置为true，则它将取代dfs.data.transfer.protection的设置，并强制所有连接必须使用专门的加密SASL握手。对于与在特权端口上侦听的DataNode的连接，将忽略此属性。在这种情况下，假定特权端口的使用建立了足够的信任。</description>    
　　</property>

    <property>
        <name>dfs.http.policy</name>
        <value>HTTPS_ONLY</value>
        <description>确定HDFS是否支持HTTPS(SSL)。默认值为"HTTP_ONLY"(仅在http上提供服务),"HTTPS_ONLY"(仅在https上提供服务,DataNode节点设置该值),"HTTP_AND_HTTPS"(同时提供服务在http和https上,NameNode和Secondary NameNode节点设置该值)。</description>    
　　</property>

    <!-- journalnode -->
    <property>
        <name>dfs.journalnode.kerberos.principal</name>
        <value>jn/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>dfs.journalnode.keytab.file</name>
        <value>/opt/keytabs/keytab</value>
    </property>
    <property>
        <name>dfs.journalnode.kerberos.internal.spnego.principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
    </property>
    
```

#### ssl-server.xml修改如下内容

配置https访问证书

```xml
    <property>
      <name>ssl.server.truststore.location</name>
      <value>/home/hadoop/keystore</value>
      <description>Truststore to be used by NN and DN. Must be specified.
      </description>
    </property>
    
    <property>
      <name>ssl.server.truststore.password</name>
      <value>tc123456</value>
      <description>Optional. Default value is "".
      </description>
    </property>
    
    <property>
      <name>ssl.server.keystore.location</name>
      <value>/home/hadoop/keystore</value>
      <description>Keystore to be used by NN and DN. Must be specified.
      </description>
    </property>
    
    <property>
      <name>ssl.server.keystore.password</name>
      <value>tc123456</value>
      <description>Must be specified.
      </description>
    </property>
    
    <property>
      <name>ssl.server.keystore.keypassword</name>
      <value>tc123456</value>
      <description>Must be specified.
      </description>
    </property>
```

#### 服务重启

```shell
/usr/local/hadoop/sbin/stop-dfs.sh
/usr/local/hadoop/sbin/start-dfs.sh
```

### Yarn认证配置

```xml
     <!-- 使用以下配置参数配置Kerberos服务主体 -->
    <property>
        <name>yarn.resourcemanager.principal</name>
        <value>rm/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>yarn.resourcemanager.keytab</name>
        <value>/op/keytabs/keytab</value>
    </property>
    
    <property>
        <name>yarn.resourcemanager.webapp.spnego-principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.spnego-keytab-file</name>
        <value>/op/keytabs/keytab</value>
    </property>

    <property>
        <name>yarn.nodemanager.principal</name>
        <value>nm/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>yarn.nodemanager.keytab</name>
        <value>/op/keytabs/keytab</value>
    </property>

    <property>
        <name>yarn.nodemanager.webapp.spnego-principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>yarn.nodemanager.webapp.spnego-keytab-file</name>
        <value>/op/keytabs/keytab</value>
    </property>

    <property>
        <name>yarn.web-proxy.principal</name>
        <value>HTTP/_HOST@HADOOP.COM</value>
    </property>
    <property>
        <name>yarn.web-proxy.keytab</name>
        <value>/op/keytabs/keytab</value>
    </property>

```

### HADOOP服务重启

```shell
/usr/local/hadoop/sbin/stop-all.sh
/usr/local/hadoop/sbin/start-all.sh
```

### 通过kerberos认证提交spark on yarn程序

```shell
## kerberos认证
kinit -kt /home/hadoop/all.keytab hadoop/hadoop1

## spark on yarn程序提交
spark-submit --master yarn --deploy-mode cluster --class dps.spark.starter.MissionStarter --driver-cores 2 --driver-memory 1G --num-executors 2 --executor-cores 2 --executor-memory 2G --name DEBUG_SINK_JDBC --queue default DEBUG_SINK_JDBC.jar -master yarn -url 'jdbc:mysql://192.168.11.91:30802/bdp_dps?serverTimezone=Asia/Shanghai&useSSL=false&useUnicode=true&characterEncoding=utf-8' -u root -pwd root -m DEBUG_SINK_JDBC  -p 1
```

------

## Dolphinscheduler认证配置

### 开启kerberos

配置认证信息`common.properties` (`api-server,worker-server`)

```shell
# whether to startup kerberos
hadoop.security.authentication.startup.state=true

# java.security.krb5.conf path
java.security.krb5.conf.path=/etc/krb5.conf

# login user from keytab username
login.user.keytab.username=hadoop/hadoop1@HADOOP.COM

# login user from keytab path
login.user.keytab.path=/home/hadoop/all.keytab

# kerberos expire time, the unit is hour
kerberos.expire.time=24
```

由于需要用linux的bidata用户执行任务，因此需要创建bidata用户的kerberos认证，方法同上，因为kerberos认证有有效期，保证任务和定时任务不失败，需要通过crontab创建定时认证

```shell
crontab -e
58 23 * * * kinit -kt /home/hadoop/all.keytab hadoop/hadoop1@HADOOP.COM
```

定时任务在所有worker主机上均需要设置，因为dolphinscheduler的任务执行默认是随机分配的

### NameNode HA

如果Hadoop集群的NameNode配置了HA的话，需要开启HDFS类型的资源上传，同时需要将Hadoop集群下的core-site.xml和hdfs-site.xml复制到worker-server/conf以及api-server/conf，非NameNode HA跳过次步骤。

拷贝`/usr/local/hadoop/etc/hadoop`路径下`core-site.xml`,`hdfs-site.xml`至dolphincheduler对应路下 (`api-server`,`worker-server`)

### 任务执行

按正常流程上传对应jar包，配置工作流定义并启动工作流即可。



## 参考资料

cdh+dolphinscheduler开启kerberos：

https://blog.csdn.net/SmellyKitty/article/details/12879235

Unable to obtain password from user的一种可能：

https://blog.csdn.net/zz_aiytag/article/details/105067703

Dolphinscheduler资源中心配置详情：

https://dolphinscheduler.apache.org/zh-cn/docs/3.1.4/guide/resource/configuration

数仓用户认证 Hadoop Kerberos配置：

https://blog.csdn.net/weixin_45417821/article/details/122759955

Hadoop in Secure Mode：

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SecureMode.html#End_User_Accounts

Kerberos 安装和使用：

https://www.jianshu.com/p/032cc462bbca

kerberos配置dolphinscheduler：

https://blog.csdn.net/m0_37759590/article/details/131338837

使用Kerberos保护Hadoop集群：

https://www.cnblogs.com/yinzhengjie2020/p/13655547.html

