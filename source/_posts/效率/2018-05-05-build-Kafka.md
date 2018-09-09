---
author: ivyxjc
date: 2018-05-05
title: Kafka 认证配置
category: 效率
tags: [kafka]
keywords:
description: 本文主要介绍如何配置Kafka认证
toc: true
---


Kafka支持多种安全验证方式，本文主要介绍使用用户名/密码的方式的方式认证Kafka。

### Zookeeper 相关更改

`zookeeper-server-start.bat`（win）
添加

```bat
IF ["%KAFKA_SASL_OPTS%"] EQU [""] (
    set KAFKA_SASL_OPTS=-Djava.security.auth.login.config=filepath\kafka_2.11-1.1.0\kafka_zoo_jaas.conf
)
```

`kafka_zoo_jaas.conf`
```
Server {  
  org.apache.kafka.common.security.plain.PlainLoginModule required  
    username="zoo-admin"
    password="zoo-admin-pass"
    user_zooUser="zooUser-pass";
}; 
```

## Kafka配置 相关更改

`kafka-server-start.bat`(win)
添加
```bat
IF ["%KAFKA_SASL_OPTS%"] EQU [""] (
    set KAFKA_SASL_OPTS=-Djava.security.auth.login.config=filepath\kafka_2.11-1.1.0\kafka_server_jaas.conf
)
```

`kafka_server_jaas.conf`

```
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="user"
    password="user-pass"
    user_kafkaUser="kafkaUser-pass"
};
    
Client {
    org.apache.kafka.common.security.plain.PlainLoginModule required
        username="zooUser"
        password="password";
};  
```

`KafkaServer`中的内容是Producer，Consumer连接Kafka时要输入的验证信息。

`Client`中的内容即连接zookeeper是需要输入的验证信息，和`kafka_zoo_jaas.conf`中的内容一致。

### 使得JVM参数生效

`zookeeper-server-start.bat`和`kafka-server-start.bat`最终都是使用`kafka-run-class.bat`来真正运行的，所以需要将这两个文件添加的JVM参数添加到此文件。

打开`kafka-run-class.bat`(win)

将`KAFKA_SASL_OPTS`添加到`set COMMAND`后面（不要放在`%Java%`之前）。


### 注意点
1. zookeeper的登陆名最好不要和kafka的登陆名有重复，在我尝试的时候发现，名称相同的时候，会直接使用KafkaServer下的用户去登陆zookeeper导致无法登陆成功。
2. 理论上讲`kafka_server_jaas.conf`中Client中为连接zookeeper的认证信息。但是在我尝试的时候发现，只在Client中记录登陆信息并不能成功，还需要将该信息填入KafkaServer中才能成功（即使如此，也未必能成功）。<br />目前发现的一个成功的组合，是使用`kafka_zoo_jaas.conf`中的username后面的用户名称，并在该文件下，添加`user_...=password`，使用此作为密码，并将这一认证信息添加到`kafka_serve_jass.conf`中的`KafkaServer`和`Client`中。